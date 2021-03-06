# Kinect Depth Image Smoothing
- "Kinect Smoothing" helps you to smooth and filter the Kinect depth image and trajectory data.

## How to Use it 
* Installation
```
git clone https://github.com/intelligent-control-lab/Kinect_Smoothing.git
pip install -r requirements.txt
```
* Running

  Please check [example.ipynb](example.ipynb) and [kinect_preprocess_example.py](kinect_preprocess_example.py).
  
 * Data preparation     
 We saved  many frames of depth images in `data/sample_img.pkl` and saved corresponding frames of position coordinates in `data/sample_pose.pkl `  
`e.g.  sample_img = [ [ image_1 ] ,  [ image_2 ], ... , [ image_t ] ].` each `image_i` has a shape of `(width, height)`.  
In case if anyone wants to use it for multiple files, then the code below should work.
```
import joblib 
import cv2
import glob 

X_data = []
file_lists = glob.glob("/*.bmp")  # image path
for files in sorted(file_lists):
    if files.endswith(".bmp"):
        image = cv2.imread(files)
        X_data.append(image)
        joblib.dump(X_data, 'image_frames.pkl')
```

## Features

1 . Depth Image Smoothing
* Hole Filling Filter

  In the original Kinect depth image, there are many invalid pixels (they appear black on the image and are registered as 0's). This function helps you to fill the invalid values with based on the valid pixels in the vicinity. The methods for hole filling are as follows:
  * "min": Fill in with the neighboring minimum valid value
  * "max": Fill in with the neighboring maximum valid value, refer to [A computationally efficient denoising and hole-filling method for depth image enhancement](https://webpages.uncc.edu/cchen62/SPIE2016.pdf)
  * "mean": Fill in with the average of all neighboring valid values
  * "mode": Fill in with the mode of neighboring valid values, refer to [Smoothing Kinect Depth Frames in Real-Time](https://www.codeproject.com/Articles/317974/KinectDepthSmoothing)
  * "fmi": Fast Matching Inpainting, refer to  [An Image Inpainting Technique Based on the Fast Marching Method](https://www.rug.nl/research/portal/files/14404904/2004JGraphToolsTelea.pdf) 
  * "ns": Fluid Dynamics Inpainting, refer to  [Navier-Stokes, Fluid Dynamics, and Image and Video Inpainting](https://conservancy.umn.edu/bitstream/handle/11299/3607/1772.pdf?sequence=1)

* Denoising Filter

  After pixel filling, Denoising filters can be used to improve the resolution of the depth image. The methods for denoising filling are as follows:
  * "modeling": filter with Kinect V2 noise model, refer to [Modeling Kinect Sensor Noise for Improved 3D Reconstruction and Tracking](http://users.cecs.anu.edu.au/~nguyen/papers/conferences/Nguyen2012-ModelingKinectSensorNoise.pdf)
  * "modeling_pf": another Kinect V2 noise modeling by Peter Fankhauser [Kinect v2 for Mobile Robot Navigation: Evaluation and Modeling](https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/104272/1/eth-48073-01.pdf)
  * "anisotropic": smoothing with anisotropic filtering [Scale-space and edge detection using anisotropic diffusion](https://authors.library.caltech.edu/6498/1/PERieeetpami90.pdf)
  * "gaussian": smoothing with Gaussian filtering

2 . Trajectory Smoothing
* Crop Filter:

  The x, y coordinates of the trajectory were captured by some object detection algorithms (e.g. Openpose). Sometimes the object will be positioned on the background,  and the depth coordinates might register as invalid values on the Kinect. The Crop-Filter crops the invalid value and run some interpolation methods to replace it. The methods for invalid value replacement are as follows:
  * conventional interpolation methods: such as "zero","linear","slinear","quadratic","cubic","previous","next","nearest".
  * "pchip": PCHIP 1-d monotonic cubic interpolation, refer to [Monotone Piecewise Cubic Interpolation](https://epubs.siam.org/doi/pdf/10.1137/0717021?casa_token=IcEKTOT2mfgAAAAA:Ymwhtl0E5xdPakjEyhIuTAS5R5MQKUu3JrdLeo1Lu0qU8IMtDoX99RGwU2Ll4saxj68nVpLaVLQ)
  * "akima": Akima 1D Interpolator, refer to [A new method of interpolation and smooth curve fitting based on local procedures](http://200.17.213.49/lib/exe/fetch.php/wiki:internas:biblioteca:akima.pdf)
  
* Gradient Crop Filter:  
  
    Similar to Crop-Filter, the GradientCrop_Filter crops the large gradient values between near pixels maybe miss-labeled as background. The methods for invalid value replacement are as follows:
  * conventional interpolation methods: such as "zero","linear","slinear","quadratic","cubic","previous","next","nearest".
  * "pchip": PCHIP 1-d monotonic cubic interpolation.
  * "akima": Akima 1D Interpolator.
  

* Smooth Filter:

  Smooth the data with a specific filter, which is effectively reducing the anomaly in the trajectory series. 
  The methods for smoothing filter are as follows:
   * "kalman": smooth the signal with Kalman filter, refer to [Fundamentals of Kalman Filtering](http://iaac.technion.ac.il/workshops/2010/KFhandouts/LectKF1.pdf)
   * "wiener": smooth the signal with Wiener filter
   * "median":  smooth the signal with median filter
   * "moving_average":  smooth the signal with moving average filter
   * "exponential_moving_average":  smooth the signal with exponential moving average filter

* Motion Sampler:

  Motion detection and filtering out the stationary part of the trajectory. (Significant movements are preserved.)

3 . Real World Coordinate Calculation
* Coordinate Calculator

  Transforming the pixel level coordinate of Kinect image to the real world coordinate. 
  		
