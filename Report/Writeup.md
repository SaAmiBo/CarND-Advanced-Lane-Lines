
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

[image1]: ./camera_cal_02.png "Undistorted"
[image2]: ./color_and_gradiant.png "ColorGradiant"
[image3]: ./perspective.png "Perspective"
[image4]: ./warped_binary.png "WarpedBinary"
[image5]: ./histogram.png "Histogram"
[image6]: ./slide_search.png "SlideSearch"
[image7]: ./poly_search.png "PolySearch"
[image8]: ./output.png "Output"
[image9]: ./curvature_equation.png "curvature_equation"

---
## Pipeline Structure
### 1. Camera Calibration 
The code for the camera calibration is located in "./Code/Calibration.ipynb". The chessboard used in the example images has 9x6 inner corners. Points associated with these corners will be object points, `objpoints`, which are the same for all images since the same chessboard has been used. The object points then are consider to be the array of [x,y,0] where x is from 0 to 8 and y from 0 to 5. 
For the image points, `imgpoints`, which are the 2D points on the chessboard image, I have used `cv2.findChessboardCorners()`. I have looped this command on all the example images and appended the associated points into an array. 
Finally, I find the calibration parameters of the camera, `mtx` and `dist`, by passing the `objpoints` and `imgpoints` to the `cv2.calibrateCamera()`. Then, I use the calibration parameters to undistort the image using `cv2.undistort()`. The result on one of the example images is shown below
![alt text][image1]

Finally, I save the `mtx` and `dist` in a pickle file named `cal_info.pkl` to use these parameters later in the pipeline.

### 2. Color and Gradiant 
I am using combination of color and gradiant analysis of the image to differentiate the lanes. For the color, I first convert the image from RGB color space to HLS and then define a threshold for the Saturation channel to pick the potential lane pixels. This threshold is a hyperparameter and can be calibrated to acheive the desired performance. Besides, I am also applying the Sobel operator using `cv2.Sobel()`. With focusing only on the horizontal derivatives and defining a threshold, I then select the potential pixels. Finally, I include the pixles that pass either the color or gradient threshold check to form the binary image. I have included the result for an example image below. Pixels from saturation threshold check are shown in red and those from the gradiant check shown in green.
![alt text][image2]

### 3. Perspective Transform
Next step would be warping the image based on the perspective transform to get a bird-eye view of the road ahead. To do this, first I need to obtain the perspective transform matrix.I pick four points on the lanes to form a quadrilateral. Then, I pick the four points I want these points to be transformed to. At this point, I use `cv2.getPerspectiveTransform()` to obtain the transform matrix and its inverse. Result of warping an example image using the transform matrix and `cv2.warpPerspective()` is shown below.
![alt text][image3]

I save the transfrom matrix, `M`, and its inverse, `Minv`, in a pickle file, `perspective_info.pkl` to use it later in the pipleline. The code for all these steps is inside the "./Code/PerspectiveTransform.pynb" file.

This warping will be applied on the binary image which is the result of previous section. An example is shown below
![alt text][image4]

### 4. Lane Finding
After getting the warped binary image, I sum the pixel values through y axis of the image to get the histogram. I can then use the pick of the histogram to find out the possible location for the right and left lane. 
![alt text][image5]

To fine the lane pixles, I define two rectangles on each side based on the base values from the histogram and slide it across the y axis of the image. I call this method the "Slide Search" The width, height, and number of pixels to recenter the window are the hyperparameters to tune. 
![alt text][image6]

Once the lane pixels are identified for one frame, the polynomial can be fit to the indices of these pixels using `np.fitpoly()`. This polynomial can then form the base around which the lane pixels can be searched. I call this method the "Poly Search".
![alt text][image7]

## Line Class
Both the left and right lanes are the instances of the defined Line class. There are a few attributes to this class. Examples of those are `Line.recent_fit` to hold the recent n number of polynomial fits to the lane, `Line.radius` as the lane radius for the current fit, and `Line.allx` and `Line.ally` as the x and y indices of the identified lane pixels. 
There are also three defined functions for this class. `Line.x_append()` and `Line.fit_append()` appends the current x values for the fitted polynomial as well as the fit parameters to the associated lists. `Line.update()` updates some of the attributes such as the radius. 

## Calculating Curvature
I have calculated the curvature of both the right and left lanes using parabola fit parameters using the equations described in the following image:

![alt text][image9]

The A and B are the first two fit parameters and xm and ym are the ratio of meter to pixel. I have used xm = 70/720 and ym = 3.7/480.
## Checking the Fit
`fit_check()` functions receives the left and right fit and does a few checks to find out if the fits are valid. To check if the lanes are almost parallel, the check is 

`if np.max(right_fitx - left_fitx) > 600 or np.min(right_fitx - left_fitx) < 350:
    flag = False`

To check if the parabolas have similar curvature, the check is as follows 

`if (10**4 * np.absolute(right_fit[0] - left_fit[0]) + (right_fit[1] - left_fit[1])) > 10:
     flag = False`
     
Whenever the `fit_check()` outputs `False` for some consecutive frames, the search method changes from the `poly_search()` to `slide_search()`.
     
## Visualize
Finally, `visualize()` function gets the warped image and the Left and Right Line objects and shades the identified lane space in front of the vehicle. It also displays the radius and distance from the lane center on the output image. Distance from the lane center is calculated assuming that the camera is mounted at the center of the vehicle and has the same heading as the vehicle. 

## Output
The output image of the pipeline is shown below:
![alt text][image8]

The code for the pipeline is included in the inside the "./Code/Pipeline_Final.pynb" file.

## Future Work
* Currently, the developed pipleline works well for the project_video.mp4 file. It smoothly identifies the lanes and shades the detected lane in front of the vehicle. However, it does not work for neither of the challenge videos. The next task would be improving the pipeline to be more robust and perform well on the challenge videos as well.
* I need to investigate the effect of different hyper-parameters in the model such as search margins and thresholds for the binary image.
* I need to make the slide_search more robust since it finds out the lanes initially and if that goes wrong, it will have cascading effects on the lane search in the next frames.
* Even though I have focused on the Saturation channel, still the different lighting conditions adversely affects the performance. It is definitely noticeable in the segments that part of the road is covered with shadows. I need to develop robust scheme to handle different lighting conditions. 