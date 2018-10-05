# CarND_Advanced_Lane_Lines

## Writeup Advanced Lane Finding Project 2
  

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README
 
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./Calibration.ipynb"  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

[im01]: ./output_images/Calibration Images.png 


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
Calibrated Test Image.png The most significant change after applying the distortion correction was around the edges of the images, likeley due to the lens of the camera. Using the camera matrix and distortion coefficient from the calibrate camera function, I applied the cv2.undistort function on the image to eliminate the image distortion. 

See Calibrated Test Image.png
 
#Camera matrix
mtx = [[  1.15777818e+03   0.00000000e+00   6.67113857e+02]
 [  0.00000000e+00   1.15282217e+03   3.86124583e+02]
 [  0.00000000e+00   0.00000000e+00   1.00000000e+00]]
  
#Distortion Coefficient
dist = [[-0.24688507 -0.02373154 -0.00109831  0.00035107 -0.00259869]]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
  
First I applied S channel filter to the undistorted image under the function `undistortAndHLS()`.The S channel filter was used because as mentioned in Lesson 6 the S channel does the best job at picking up the lane lines under very different color and contrast conditions compared to R and H channels. Next I applied Sobel edge detection in the X and Y directions under the function `absSobelThresh()`, then combined the thresholds using the function combineThresholds which can be viewed under 4_S_Sobel.jpg. This provided sufficient thresholding and once I arrived at this result I decided not to use the direction of the gradient.
 
#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform was applied to the binary thresholded image under the function 'warper()'. The code for my perspective transform includes a function called `warper()`, and is based on the tranformation matrix identified in `Trans_Matrix.ipynb`.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:
 

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 455      | 200, 0        | 
| 705, 455      | 1080, 0       |
| 1130, 720     | 1080, 720     |
| 190, 720      | 200, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]  5_Perspective.png 


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Using the lessons learned from the Advanced Computer Vision Lesson, lane-line pixels were identified using the transformed binary image and using the sliding window polynomial fit technique was used to estimate the location of the lane lines. The two highest peaks in the histogram were identified as the lane lines then the window was slide upwared to identify the next row of lane lines. After itterating with many scenarios, I found that the number of windows 9, margin 110 and min pix 50 yielded near optimal results. `laneLine()`

The results of fitting the lane lines with a 2nd order polynomial can be viewed on 6_Lane_lines.Png

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature was estimated using the function `calculateCurvature`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Example image of the result plotted back down onto the road can be viewed on 7_Green_Lane.png. I implemented this step by warping the image back to the real world space using the inverse perspective matrix in the function `drawLine()`. 
 

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video/project_video.mp4)
  
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Following the strategies covered in Lessons 5, 6 and 7 I was able to successfully calibrate the camera, threshold images, transform the image, identify peaks in rows of pixels as lane lines, fit a quadratic to the peak pixels, estimate the curvature of the road based on estimating the curavture of the peak pixels.

The pipeline has breaks with detection incomming dashed lane lines. This issue can be resolved by storing the previous value and using a moving average of the lane location to reduce the sensitivity to new dashed lane lines.

The pipeline works well on flat roads, but will likely need additional processing for roads with hills and higher contrasting shadows. If we were going to pursue this project further, there would have to be another sensor that can provide lidar measurement data which provides the true distance between the vehicle and the environment. Image data alone cant tell the difference between a sharp turn and a slight turn with a hill. 
