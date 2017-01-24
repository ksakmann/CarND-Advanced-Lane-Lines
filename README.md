## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

The goals / steps of this project are the following:  

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply the distortion correction to the raw image.  
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view"). 
* Detect lane pixels and fit to find lane boundary.
* Determine curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

---
[//]: # (Image References)

[image1]: ./Undistort.jpg "Undistorted"
[image2]: ./Undistort_test5.jpg "Undistorted"
[image3]: ./binary.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

### [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
In the following I will consider the rubric points individually and describe how I addressed each point in my implementation.  
The images for camera calibration are stored in the folder called `camera_cal`.  Images in `test_images` are for testing the pipeline on single frames.  Results are in the  `ouput_images` folder and subfolders `stage0, stage1, stage2`.

## Stage 0 - Camera calibration 

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in IPython notebook located in "./stage0_camera_calibration.ipynb" .  

I start by preparing "object points", which will are (x, y, z) coordinates of the chessboard corners in the world (assuming coordinates such that z=0).  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

`objpoints` and `imgpoints` are then used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
![Undistort][image1]

## Stage 1 - Test Image Pipeline

#### 2.Provide an example of a distortion-corrected image.
Applying the undistortion transformation to a test image yields the following result (left distorted, right corrected)
![Undistort][image2]
#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.
For color thresholding I worked in HLS space. Only the L and S channel were used. I used the s channel for a gradient filter along x and saturation threshold, as well as the l channel for a luminosity threshold filter. A combination of these filters
is used in the function `binarize` in the file `stage1_test_image_pipeline.ipynb`. The binarized version of the image above looks as follows
![Binarized image][image3]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:


