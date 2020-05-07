---
title: 'Writeup'
disqus: Ansheel Banthia
---

Project 1 - Advanced Lane Finding :car: 
===

Udacity Self-Driving Car Engineer Nanodegree

## Table of Contents

 - [ Summary ](#sum)
 - [ Description of Pipeline ](#des)
      - [ Camera Calibration ](#cam)
      - [ Distortion Correction ](#dis)
      - [ Binary Threshold ](#bin)
      - [ Perspective Transformation ](#per)
      - [ Lane Detection ](#lan)
      - [ Radius of Curvature & Car Position Estimation ](#rad)
      - [ Result ](#res)
 - [ Potential Shortcomings ](#short)
 - [ Improvement to Pipeline ](#fut)

<a name="sum"></a>
## Summary

![](https://github.com/Ansheel9/P3-Collabration-Competition-DeepRL/blob/master/Images/plot.PNG)

When we drive, we use our eyes to decide where to go.  The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle.  Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

In this project you will detect lane lines in images using Python and OpenCV.  OpenCV means "Open-Source Computer Vision", which is a package that has many useful tools for analyzing images.

The goals / steps of this project are the following:

- Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
- Apply a distortion correction to raw images.
- Use color transforms, gradients, etc., to create a thresholded binary image.
- Apply a perspective transform to rectify binary image ("birds-eye view").
- Detect lane pixels and fit to find the lane boundary.
- Determine the curvature of the lane and vehicle position with respect to center.
- Warp the detected lane boundaries back onto the original image.
- Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

<a name="des"></a>
## Description of Pipeline
---
My pipeline consisted of 7 major steps. First, I performed camera callibration. Then I applied distortion correction on the images in order to remove distortion due to 3d to 2d conversion. Third, I created binary transform image to detect the sharp edges. Fourth, I apply a perspective transform to get birds-eye view. Fifth, I found lane pixels by dividing the lane into 2 part, left & right. Sixth, I found curvature of the lane & vehicle position with respect to tne centre. Lastly, I warp the detected lane boundaries as well as numerical estimation on the original image. Brief description of each step is given as follow:

<a name="cam"></a>
### Camera Calibration

OpenCV functions <code> findChessboardCorners() </code> and <code> drawChessboardCorners() </code> were used to automatically find and draw corners in an image of a chessboard pattern. "Object points", which will be the (x, y, z) coordinates of the chessboard corners in the real world were prepared assuming the chessboard pattern is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, <code> objp </code> is just a replicated array of coordinates, and <code> objpoints </code> is appended with a copy of it every time <code> findChessboardCorners() </code> successfully detect all chessboard corners in a test image. <code> imgpoints </code> is appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard pattern detection. It is expected to detect 9x6 grid of corners on the calibrated images.

![](https://github.com/Ansheel9/P3-Collabration-Competition-DeepRL/blob/master/Images/plot.PNG)

<a name="dis"></a>
### Distortion Correction

To measure distortion of a camera it is possible to use photos of real world object with well-known shape.

The output `objpoints` and `imgpoints` were used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. It returns the camera matrix (`mtx`), distortion coefficients (`dist`), rotation and translation vectors, etc. For the code, see the second code cell of the project Jupyter notebook. A function `undistort(img)` was also defined. This functions correct distortion of a given image using the `cv2.undistort()` function and previously computed camera matrix and distortion coefficients.

![](https://github.com/Ansheel9/P3-Collabration-Competition-DeepRL/blob/master/Images/plot.PNG)

<a name="bin"></a>
### Binary Threshold

In this section, a binary image is created. I used a combination of color and gradient thresholds to generate a binary image. The implementation extracts the HLS colorspace, where especially the S channel seems to highlight the lane lines quite well. A Sobel filter was applied followed by a custom thresholding to select relevant pixels. This was followed by a thresholding of the S channel of HLS. Also, directional & magnitude gradent thresholding was performend to highlight the lane lines. Finally, all binary output were combined to produce the final output.

![](https://github.com/Ansheel9/P3-Collabration-Competition-DeepRL/blob/master/Images/plot.PNG)

<a name="per"></a>
### Perspective Transformation

A perspective transform maps the points in a given image to different, desired, image points with a new perspective. In our case the new perspective is a top-down view, or “birds-eye” view that lets us view a lane from above. This transform is particularly important when measuring the curvature of the lines.  

 In order to perform the transform I defined four source points that form the trapezoid in the right side image below and four destination points of where I want the source points to appear after the transformation, or warp. 

|                       | Source        | Destination | 
|:---------------:|:-------------:|:-------------:| 
| Bottom Left   | 580.0, 460.0     | 200.0, 0     | 
| Bottom Right | 270.0, 670.0   | 200.0, height   |
| Top Right       | 1100.0, 670.0     | width - 200.0, height      |
| Top Left          | 740.0, 460.0   | width - 200.0, 0        |

These source and destination points are used in the function
`getPerspectiveTransform()`to obtain the transformation matrix, M. M is then applied to the function `warpPerspective()` to warp the image to a top-down view and the destination points are then drawn on to the image. 

![](https://github.com/Ansheel9/P3-Collabration-Competition-DeepRL/blob/master/Images/plot.PNG)

<a name="lan"></a>
### Lane Detection

After applying calibration, thresholding, and a perspective transform to the image, I have a binary image where the lane lines stand out clearly, but I still need to decide explicitly which pixels are part of the lines and which belong to the left line and which belong to the right line.

Two different methods were used to detect the lane lines: sliding window search and search from prior.

First, a histogram of where the binary activations occur across the image is computed and split into two sides, one for each line. The two highest peaks from the histogram provides a starting point for determining where the lane lines are.

![](https://github.com/Ansheel9/P3-Collabration-Competition-DeepRL/blob/master/Images/plot.PNG)

A sliding window is then used to move upward in the image (further along the road) to determine where the lane lines go. A second-order polynomial is also fitted to the lanes detected from the histogram as seen below. The number of sliding windows are represented by the green rectangles and the size of each window is defined by the parameter “margin”.

![](https://github.com/Ansheel9/P3-Collabration-Competition-DeepRL/blob/master/Images/plot.PNG)

The second method used was a “search from prior”. Instead of starting fresh on every frame and doing a blind search again this method will search a margin around the previous lane line position. Once the lane lines are identified in the first frame, a highly targeted search is then performed for the next frame.

<a name="rad"></a>
### Radius of Curvature & Car Position Estimation

Two functions were implemented for measuring the lane curvature and vehicle postion with respect to the center of the lane.

The function `measure_curvature_real()` takes in the pixel values for the left and right lane line. Then the pixel-to-meter ration `ym_per_pix` and `xm_per_pix` are defined to calculate the real world units transformed from pixel values, since until now everything has been pixel values. It is expected as per a standard that the lane width is approximately 3.7 meters and the tracking lenght of the lane is 30 meters.

* `ym_per_pix` = 30/720 # meters per pixel in y dimension
* `xm_per_pix` = 3.7/700 # meters per pixel in x dimension

The function `measure_offset_to_center()` calculates the offset to the center of the lane by taking the two fitted polynomials for each lane line and finding the x-coordinate where they intersect with the bottom of the image frame, i.e. at y_max. The center is then calculated by finding the center point between this two x-coordinates. Finally, the value is transformed from pixel-values to real-world values in meters, and then returned.

![](https://github.com/Ansheel9/P3-Collabration-Competition-DeepRL/blob/master/Images/plot.PNG)

<a name="res"></a>
### Result

The final step in processing the images was to plot the polynomials on to the warped image, fill the space between the polynomials to highlight the lane that the car is in, use another perspective trasformation to unwarp the image from birds eye back to its original perspective, and print the distance from center and radius of curvature on to the final annotated image.

![](https://github.com/Ansheel9/P3-Collabration-Competition-DeepRL/blob/master/Images/plot.PNG)

<a name="short"></a>
## Potential Shortcomings with Current Pipeline
---
 - The pipeline could distort from lane lines when there are dark shadows of trees or others in the challenge videos.
 - By adding a region of interesting to the processing pipeline a lot of noise can be removed from the input image. Here a more robust system will be beneficial.

<a name="fut"></a>
## Improvement to Pipeline
---
 - The video filtering algorithm can be improved in order to make it more robust on the harder_challenge_video.mp4 and other real-world videos.
 - Use Convolutional Neural Network or other Deep Learning methods to create a semantic segmentation solution for detecting lane lines.


