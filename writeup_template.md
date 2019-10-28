
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

[image1]: ./output_images/straight_lines1_Undistorted.jpg "Undistorted image"
[image2]: ./output_images/test1_Undistorted.jpg "Road Undistorted image"
[image3]: ./output_images/test4_Color_binary.jpg "Color Binary Example"
[image4]: ./output_images/test4_Combined_binary.jpg "Combined Binary Example"
[image5]: ./output_images/straight_lines1_Warped_Perspective_Transformed.jpg "Warp Example"
[image6]: ./output_images/test6_sliding_windows.jpg "Sliding windows Visual"
[image7]: ./output_images/test6_result_image.jpg "Output"
[image8]: ./output_images/test6_Finalresult_image.jpg "Final Output"
[video1]: ./output_images/output_project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./Avanced-Lane-Finding.ipynb".

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  The distortion correction to the test image was found by using the `cv2.undistort()` function with camera matrix, distortion coefficients. Please refere the code in cell 1 of Avanced-Lane-Finding.ipynb for calibrate_camera and Distort functions . Please see the below distorion corrected calibration image: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

Using the cv2.calibrateCamera() function to compute the camera calibration and distortion coefficients, and then applied this distortion correction to the test image using the cv2.undistort() function and obtained this result

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.

I used HLS space to help detect lane lines of different colors and under different lighting conditions. OpenCV provides a function hls = cv2.cvtColor(im, cv2.COLOR_RGB2HLS) that converts images from one color space to another. The S channel picks up the lines well.
It's most important that you reliably detect different colors of lane lines under varying degrees of daylight and shadow. We can clearly see which parts of the lane lines were detected by the gradient threshold and which parts were detected by the color threshold by stacking the channels and seeing the individual components. So I created a binary combination of these two images to map out where either the color or gradient thresholds were met. Here's what that looks like in code (cells 6 and 7 of the IPython notebook located in "./Avanced-Lane-Finding.ipynb"):
The output is shown below. The final image color_binary is a combination of binary thresholding the S channel (HLS) and binary thresholding the result of applying the Sobel operator in the x direction on the original image.


![alt text][image3]
![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called Warped_image(), which appears in 8th cell in Avanced-Lane-Finding.ipynb notebook. The Warped_image() function takes an image (img) as input. I chose to hardcode the source and destination points in the following manner:

src = np.float32([[170,720],[560,460],[706,460],[1130,720]])
dst = np.float32([[200,720],[200,0],[1080,0],[1080,720]])

This resulted in the following source and destination points:

Source	Destination
190, 720	200, 720
578, 460	200, 0
706, 460	1080, 720
1130,720	1080, 720

Using cv2.getPerspectiveTransform(src, dst), perspective transform the image is found and using cv2.warpPerspective(img, M, img_size, flags=cv2.INTER_LINEAR), warped image was found.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I took a histogram along all the columns in the lower half of the image and split the histogram into two sides, one for each lane line. Then used the two highest peaks from histogram as a starting point for determining where the lane lines are, and then use sliding windows moving upward in the image (further along the road) to determine where the lane lines go. Fitted a polynomial to all the relevant pixels found in your sliding windows in fit_polynomial(). Code cells 10, 11,12 and 13 of the IPython notebook located in "./Avanced-Lane-Finding.ipynb" are used for histogram and finding lane lines, fit polynomial.

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code is in 14,15,16,17 cells of the IPython notebook located in "./Avanced-Lane-Finding.ipynb". I defined the conversions in x and y from pixels space to meters and assumed the camera is mounted at the center of the car, such that the lane center is the midpoint at the bottom of the image between the two lines detected. The offset of the lane center from the center of the image (converted from pixels to meters) is the distance from the center of the lane.
Then I used cv2.putText() to put the measurment onto the image as below:

![alt text][image7]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 18,19,20 code cells in Avanced-Lane-Finding.ipynb file in the function Plot_Lane_Lines_Single_Image(). Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result] [video1](./output_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I think the solution in general is not optimal for general lane-finding. As seen in the challenge videos, changes in road conditions and colors present a problem and must have specific color filters. Also, other lines are picked up by the pipeline that match all the processing but are not the lane lines. Similarly, during lane-change, lane-merging or any other situation where lanes are converging to normal pattern, the algorithm will fail.
Finally, many roads don't have lane markings, or the markings are very faint or broken up so determining a path would fail as well.
In order to better deal with varying lightning conditions, I will explore the deep learning techniques to find the lane lines. I think deep learning will work better if I collect enough good data.  
