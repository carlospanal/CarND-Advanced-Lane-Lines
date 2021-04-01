# **Finding Lane Lines on the Road** 


---

Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


[image1]: ./camera_cal/calibration3.jpg "output"
[image2]: ./camera_cal_output/out_calibration3.jpg "output"
[image3]: ./test_images/straight_lines1.jpg "output"
[image4]: ./output_images/test_calibrated_output/out_undis_straight_lines1.jpg "output"
[image5]: ./test_images/test3.jpg "output"
[image6]: ./output_images/test_binary_output/out_binary_test3.jpg "output"
[image7]: ./output_images/test_binary_output/out_binary_test3.jpg "output"
[image8]: ./output_images/test_perspective_output/out_perspective_test3.jpg "output"
[image9]: ./output_images/test_windows_output/3_gray.jpg "output"
[image10]: ./output_images/test_windows_output/3_window.jpg "output"
[image11]: ./output_images/test_windows_output/3_histogram.jpg "output"
[image12]: ./output_images/test_windows_output/3_search_around.jpg "output"
[image13]: ./output_images/test_info/info_example.JPG "output"

---

### Reflection

### 1. Calibration

I used a set of chessboard images in order to get the calibration parameters of the used camera.
Defined function: find_image_distortion()  
Main functions used inside find_image_distortion():  
    * cv2.findChessboardCorners()  
    * cv2.calibrateCamera()
    
Effect on chessboard image after applying cv2.undistort():

![alt text][image1]  ![alt text][image2]  

Effect on training image after applying cv2.undistort():

![alt text][image3]  ![alt text][image4]  


### 2. Binary image

This pipeline uses an undistorted image as an input and changes its color space from RGB to HLS and Lab spaces.  

L channel of HLS space is used to detect white lanes, while b channel of Lab space is used to detect yellow lanes.

Defined function: binary_pipeline()  
Main functions used inside find_image_distortion():  
    * cv2.cvtColor()
    
Effect on training image after applying binary threshold:
    
![alt text][image5]  ![alt text][image6]  

### 3. Perspective change

In order to get a view of the real dimensions of the lanes, we have to apply a perspective change.  
I defined get_transform_matrix(). This function takes a list of images with straight lanes and uses canny algorithm and hough transform to detect them. I found interesting to do it this way instead of "hardcoding" origin and destination points.  
Then, the mean value of start/end point of the lanes is calculated and used with cv2.getPerspectiveTransform() to get a perspective transformation matrix M, as well as the inverse transformation matrix Minv.

Then, cv2.warpPerspective() uses M and a binary image in order to create a binary top-down view.
Example:

![alt text][image7]  ![alt text][image8]  

    
### 4. Detect lane pixels and fit polynomial

First, I defined hist() to generate an histogram of the binary warped image


![alt text][image9]  ![alt text][image10]  

Then, peaks of the histogram are used to set an starting point for the sliding window method.  
I defined find_lanes() for this method:  
It defines a set of "windows" with a certain margin. The first window detects lane points that are close to it.  
Next windows change their X position based on the position of pixels of the previous window. In addition, I added a new functionality that stores the last x-position change of last windows in order to keep that change even when a new window doesn't find any pixel.  

I also defined fit_polynomial() that takes a set of pixels and fits a second order polynomial to it:  
![alt text][image11]

For the following frames, i defined search_around_poly() that searchs pixels around the previous polynomial based on a certain margin:  
![alt text][image12]


### 5. Measure curvature of lanes and offset of car

I defined measure_and_paint_offset_real() in order to calculate the deviation of the car from the center of the lane.  
Position of the car is considered to be at the center of the image. Center of the lane is calculated as the mean value of the closests points to camera of left and right lines.

I defined measure_curvature_real() in order to calculate curvature of left and right lines. This calculation is based on A and B parameters of the fitted polynomial lines.

Both functions use real world data in order to translate values from pixels to meters.  

(Result is shown in the image of the next section)

### 6. Inversed perspective change

inv_transform() was defined in order to translate the fitted polynomials to original images.
It draws color between both polynomials and uses cv2.warpPerspective and cv2.addWeighted to unwarp the result and combine it with the image.


![alt text][image13]

### VIDEO PIPELINE

process_image() has been defined in order to process the video.
It follows the same order as previous explained sections, and uses some global variables in order to store values between frames.
"counter" variable is used to check if the first frame is being processed, so the sliding window method is applyed only in that case.

The pipeline asumes that the following parameters have already been calculated and are previously existing:  
mtx, dist, M, Minv  
The reason for this is that these are early calculations of the project that slow down the development of the rest of the pipeline.
  

## Outputs:

[link to folder containing video on its final state](videos_output)  


## Discussion:

### Potential shortcomings

  
* This process is not considering the fact that there could be a much closer car on the same lane.  
* Lane changing is also not considered on this model.  
* Different line colors would break the pipeline.  


### Possible improvements

* Apply smoothing by averaging previous frame parameters. 
* Use deep learing techniques to detect lines.
* Use some method to reject inconsistent frame




