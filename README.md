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
[image4]: ./birdseye.jpg "Warp Example"
[image5]: ./roi.jpg "Region of interest"
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
A perspective transform to and from "bird's eye" perspective is done in a function called `warp()`, which appears in 5th cell of the notebook `stage1_test_image_pipeline.ipynb`.  The `warp()` function takes as input an color image (`img`), as well as the `tobird` boolean paramter. The parameters `src` and ` dst` of the transform are hardcoded in the function as follows:

```
    corners = np.float32([[190,720],[589,457],[698,457],[1145,720]])
    new_top_left=np.array([corners[0,0],0])
    new_top_right=np.array([corners[3,0],0])
    offset=[150,0]
    
    img_size = (img.shape[1], img.shape[0])
    src = np.float32([corners[0],corners[1],corners[2],corners[3]])
    dst = np.float32([corners[0]+offset,new_top_left+offset,new_top_right-offset ,corners[3]-offset])    
```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 190, 720      | 340, 720    | 
| 589, 457      | 340, 0      |
| 698, 457      | 995, 0      |
| 1145,720      | 995, 720    |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. See the following image
![alt text][image4]
Additionally, I included a region of interest that acts on the warped image to reduce artefacts at the bottom of the image.
This region is defined through the function `region_of_interest()` and was tested using the wrappers `warp_pipeline(img)` and `warp_binarize_pipeline(img)`An example is shown below. 
![alt text][image5]


####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
The function `find_peaks(img,thresh)` in cell of `stage1_tes_images_pipeline.ipynb` takes the bottom half of a binarized and warped lane image to compute a histogram of detected pixel values. The result is smoothened using a gaussia filter and peaks are subsequently detected using. The function returns the x values of the peaks larger than `thresh` as well as the smoothened curve. 

Then I wrote a function `get_next_window(img,center_point,width)` which takes an binary (3 channel) image `img` and computes the average x value `center` of all detected pixels in a window centered at `center_point` of width `width`. It returns a masked copy of img a well as `center`.

The function `lane_from_window(binary,center_point,width)` slices a binary image horizontally in 6 zones and applies `get_next_window`  to each of the zones. The `center_point` of each zone is chosen to be the `center` value of the previous zone. This way the windows can follow the lane line pixels if the road bends. The final result is a masked image of a single lane line. 
seeded at `center_point`. An example is shown below. 
![alt text][image6]


Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image7]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image8]
