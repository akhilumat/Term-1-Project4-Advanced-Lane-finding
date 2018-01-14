
## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[//]: # (Image References)

[image1]: output_images/Capture.jpg "Image with Lane lines detected"
[image2]: output_images/Capture2.jpg "Histogram"
[image3]: output_images/Capture3.jpg "Binary Warped Image"
[image4]: output_images/Capture4.jpg "Warp Example"
[image5]: output_images/Capture5.jpg "Lines drawn to get source points for warping"
[image6]: output_images/Capture6.jpg "Binary output"
[image7]: output_images/Capture7.jpg "Undistorted road Image"
[image8]: output_images/Capture8.jpg "Distorted chessboard image"
[image9]: output_images/Capture9.jpg "Undistorted chessboard image"
[image10]: output_images/Capture10.jpg "Points drawn on chessboard"
[video1]: project4_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

Firstly I defined 'objectpoints' for 9x6 chessboard piece. Imagepoints were obtained using 'cv2.findChessboardCorners' and sample camera calibration images. 'cv2.drawChessboardCorners' function was used to draw points on the chesboard. 

![alt text][image10]

Then I defined a cal_undistort fuction which took image, objectpoints and imagepoints as input and returned a undistorted image

![alt text][image8]

![alt text][image9]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image7]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

To create a threshhold binary image I used HLS threshhold, x gradient threshold and direction threshhold. Threshhold limit for x gradient was set from 20-100, direction threshhold was set from 0.7-1.5 and Saturation threshhold was set from 170-255. Outputs of all three were combined to get the final binary output.

![alt text][image6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

For perspective transform, I started by finding source points. Two lines on the undistorted image of the road was drawn to get the end points of the polygon. This can be seen from the image below.

![alt text][image5]

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 315, 700      | 400, 0        | 
| 570, 470      | 400, 720      |
| 1170, 700     | 1000, 720      |
| 725, 470      | 1000, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I use histogram to find the lane lines. Firstly I drew the historam of the binary warped image, and then found the columns of maximum values to get leftx base and rightx base. 

![alt text][image2]

9 windows were used to get the right and left lane indices by moving the window from top to bottom of image. These indices were used to get leftx lefty and rightx righty points, and these points were used to polyfit the curve. After finding left_fit and right_fit values, I used these values for the second frame. After first frame, the indices of next frames were found only in the vicinity of the previous detected lines. Only if the indices were less in number search window function was used. This made code to run quickly since unnecessary iterations were avoided. All the left_fit and right_fit values were appended to the left_i and right_i matrix, so that we can find average of last 9 frames. Then ploty was defined with all the y values of the image and corresponding x values for left and right lanes were calculated. These points were then used to make a combined image with lane lines and undistorted image. 

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate the radius of curvature, I took one Y value at the bottom of image. To get values in meter, meter per pixel for x and y dimension was defined. ym_per_pix = 30/720 and xm_per_pix = 3.7/700 was used. Then I multiplied these values with leftx lefty and rightx righty values, and polyfitted a curve on those points. Using Y value and polyfit curve values, I calculated left and right line curvature. Mean of these values was taken to get road curvature. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here is an example of my result on a test image:

![alt text][image1]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](project4_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

It took me lots of time to figure out the threshhold values of the binary output functions. Pipeline was failing where brightness of the video increased too much. I played around with threshhold for saturation, x gradient and direction to get the desired output. To make it more robust we can use more color channels and add there contribution to the combined binary image. We can also improve our lane finding algorithm by using some method other than using histogram.
