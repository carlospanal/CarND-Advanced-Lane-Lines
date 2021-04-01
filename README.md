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

    
### 3. Perspective change
I created 2 pipelines. 
They work the same way, but the first one takes a list of images as an input instead of a single image and,
unlike the definitive pipeline, its region of interest is specifically designed for the size of training images.  
  
The first one was used for training images and includes a widget that lets you control the hough function parameters 
and also lets you move through the list of images.  
  
Initial values for parameters of this widget are the same as the ones chosen for both pipelines, and its use
doesn't affect the stored output of the involved training images.  

The definitive pipeline takes an image as an input and consists on the following steps:  

  1 - Convert intial RGB image to grayscale using grayscale()  

  2 - Apply smoothing to the image using gaussian_blur()  

  3 - Use Canny algorithm on the image to detect edges using canny()  

  4 - Define a region of interest, depending on the image size and using  np.array()  

  5 - Apply defined region to the output of canny() function using region_of_interest()  

  6 - Calculate Hough Lines using hough_lines()  
  
After creating this definitive pipeline, i started modifying draw_lines() so that function
was able to draw 1 averaged line for each lane instead of all the detected hough lines.
Basic steps of this strategy:  

  1 - Group lines as left or right lines depending on their slope  
  2 - Use polyfit() and poly1d() to fit a line through each group of points  
  3 - Set starting and end points of each averaged lane considering the fitted line and image dimensions  
  
Regarding the extra challenge, I partially completed it by making these modifications:  

  1 - Define region of interest based on image dimensions instead of "hardcoding" it  
  2 - Define 2 global variables that store last averaged lines so the program doesn't break  
      when there is frame with no detected lines in some of the lanes  
  3 - Set a minimal slope so horizontal lines are discarded  
  
 The video of this challenge is still unstable from 0:04 to 0:05.  
 I tried some strategies like lowering the canny low and high threshold, but it had a negative effect on the previous videos.  

## Outputs:
Output of the pipeline when applied to a training image:  
![alt text][image1]  

[link to folder containing videos on their final state](test_videos_output)  

[link to folder containing videos of different steps of the process](other_outputs)  


### 2. Identify potential shortcomings with your current pipeline


Changes on light instensity or road colors can break this pipeline, as happens with the last challenge video.   
  
Another problem is that this process is not considering the fact that there could be a much closer car on the same lane.  
In addition, the system is not robust against cracks on the road or in the car front glass.  


### 3. Suggest possible improvements to your pipeline

Averaged lines have some kind of vibration due to the movement of the car, a PID could be easly implemented to correct that.  
  
Another potential improvement, but a bit more complicated one, could be to divide the image in order to apply canny algorithm
with different values to different parts of the image depending on local changes on road colors.  
