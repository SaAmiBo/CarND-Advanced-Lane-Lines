
# Advanced Lane Finding Project

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

[image1]: ./camera_cal_01.png "Undistorted"

---
## Pipeline Structure
### 1. Camera Calibration 
The code for the camera calibration is located in "./Code/Calibration.ipynb". The chessboard used in the example images has 9x6 inner corners. Points associated with these corners will be object points, `objpoints`, which are the same for all images since the same chessboard has been used. The object points then are consider to be the array of [x,y,0] where x is from 0 to 8 and y from 0 to 5. 
For the image points, `imgpoints`, which are the 2D points on the chessboard image, I have used `cv2.findChessboardCorners()`. I have looped this command on all the example images and appended the associated points into an array. 
Finally, I find the calibration parameters of the camera, `mtx` and `dist`, by passing the `objpoints` and `imgpoints` to the `cv2.calibrateCamera()`. Then, I use the calibration parameters to undistort the image using `cv2.undistort()`. The result on one of the example images is shown below
![alt text][image1]