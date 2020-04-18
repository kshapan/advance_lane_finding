## Writeup : Advance Lane Finding Project

---


[//]: # (Image References)

[image1]: ./output_images/undist_test1.jpg "Undistorted"
[image2]: ./output_images/combine_test1.jpg "Combined Binary"
[image3]: ./output_images/wrapped_binary_test1.jpg "Warp Example"
[image4]: ./output_images/wraped_test1.jpg "Find lane pixel"
[image5]: ./output_images/unwrapped_output_test1.jpg "Unwarp Example"
[image6]: ./output_images/output_test1.jpg "Output"
[video1]: ./test_output_videos/output_project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

### Pipeline (single images)

My pipeline consist of several steps to find lane. There are described in details as belows 

#### 1. Camera calibration and applying undistortion.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result which is distortion corrected image: 

![alt text][image1]

#### 2. Use color transforms, gradients or other methods to create a thresholded binary image. 

I have used sobel_x gradient and s_channel threshold to generate the combine binary image. Please check the function get_combined_binary() for details provided in `P2.ipynb`.  Here's an example of my output for this step.  

![alt text][image2]

#### 3. Performe a perspective transform to get a transformed image.

This step takes combine binary image fro step 2 and calculate source (`src`) and destination (`dst`) points to perform perspective transform.
Please check the function get_wrapped_binary() for details provided in `P2.ipynb`.  
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. Here is out of this step.

![alt text][image3]

#### 4. Identified lane-line pixels and fit their positions with a polynomial

In this step, I used histogram and sliding window mechanism to identify lane line pixel and the apply a 2nd degree polinomial to get the coefficient for curve across the lane pixel. Please check the functions find_lane_pixels(), get_poly_fit() and fit_poly() for details provided in `P2.ipynb`. The output of step is shown in below image. 

![alt text][image4]

#### 5. Calculate the radius of curvature of the lane and the position of the vehicle with respect to center.

In this step, I calculated curvature of the lanes and the position of the vehicle with respect to center. Please check the functions measure_curvature_real() and get_vehicle_position() for details provided in `P2.ipynb`. After this we are caluclating the radius of lane by taking mean of Left lane curavture and right lane curvature.

#### 6. Result plotted back down onto the road such that the lane area is identified clearly.

In this step, first we unwrapped the transformed image and then annoted it on the original image to get the resulted out.
Below image show unwrapped image.
![alt text][image5]

Below image show final result of the pipeline.
![alt text][image6]

---

### Pipeline (video)

Once I found the lane lines in one frame of video, I don't need to search blindly in the next frame. I can simply search within a window around the previous detection.

For example, if you fit a polynomial, then for each y position, you have an x position that represents the lane center from the last frame. Search for the new line within +/- some margin around the old line center.
For this purpose, I have used class laneBoundary class function detect_lane_pixels() details provided in `P2.ipynb`. Here, sanity check is also peformed. 
If sanity checks reveal that the  detected lane lines are problematic then I assume it was a bad or difficult frame of video, and retain the previous positions from the frame prior and step to the next frame to search again. If I lose the lines for 3 frames in a row, I start searching from scratch using a histogram and sliding window, to re-establish the measurement.


Here's a the output video:

![alt text][video1]

---

### Discussion

#### 1. Problems / issues you faced in implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There are some scenarios where the some horizontal patches were detected along the lanes. This has resulted in inclusion of these non lane pixels to get identified as lane pixels and hence it effected the output of pipie line. 
I tried several thresholds to get rid of these horizontal patches, however it was not of any use. I even tried different color thresholding but it didnt help. I guess there might be some different mechanism to get rid of this horizontal patches and still have the lanes which will make sure to give us more accurate output.   
