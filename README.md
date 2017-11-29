## Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("Bird’s-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The code for this project can found in the Ipython Notebook `Advance_line_finding_project.ipynb` that is part of this repository.

[//]: # (Image References)

[image1]: ./report-images/calibration-example.png "Calibration example"
[image2]: ./report-images/un-distortion-example.png "Calibration example"
[image3]: ./report-images/threshold-example.png "Calibration example"
[image4]: ./report-images/birds-eye-transformation.png "Bird's-eye transformation"
[image5]: ./report-images/warp-example.png "Bird's-eye transformation in curved road"
[image6]: ./report-images/transformation_pipeline.png "Transformation pipeline"
[image7]: ./report-images/line-detection-sldwnd.png "Line detection with sliding window"
[image8]: ./report-images/line-detection-assisted.png "Line detection based on previously detected lines"
[image9]: ./report-images/printed-image.png "Image with printed line, position and curvature"
[video1]: ./result.mp4 "Video" 


### Camera Calibration

#### 1. Computed the camera matrix and distortion coefficients.

Camera calibration is performed in the function `calibrate_camera()` in the 3rd code cell of the Ipython Notebook. The function goes through the following steps:
1. Creates objective points for a 9x6 grid
2. Use opencv `findChessboardCorners` function to find corners in chessboard calibration images
![alt text][image1]
3. Use opencv `calibrateCamera` function to obtain camera matrix and distortion coefficients for the camera

Once obtained this parameters they can be used to apply the distortion correction to the any image taken by this camera using the `cv2.undistort()` function.

### Pipeline (single images)

#### 1. Example of a distortion-corrected image.

The matrix and distortion parameters obtained in the previous step can be used to apply the distortion correction to the any image taken by this camera using the opencv `undistort()` function. Example:
![alt text][image2]


#### 2. Color transforms and gradients.

A combination of color and gradient thresholds is used to generate a binary image. The transformation is done in function `color_gradient_threshold` in 6th cell of the Ipython Notebook.
For the color thresholding the following color spaces and channels were used:
- RGB --> channel R
- HLS --> channel S
- HSLuv --> channel L
- Lab --> channel b

As for the gradient thresholding the sobel transformation in x direction was applied on the S channel binary image (from HLS color space).
The thresholds for each channel were adjusted after several tries to obtain a consistent mix and readjusted after processing the video. The following example show the transformation result:
![alt text][image3]


#### 3. Perspective transform and provide an example of a transformed image.

For the perspective transform an image with straight lines was used and a polygon adjusted to fit the lines. This polygon is supposed to form a rectangle in a bird’s-eye view and hence its corner points were taken as source and matched to the corner points of a rectangle

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 202, 720      | 350, 720      | 
| 1110, 720     | 950, 720      |
| 684, 450      | 950, 0        |
| 596, 450      | 350, 0        |

These points were used in the openCV `getPerspectiveTransform` function to obtain the M matrix that drives the warp transformation. This process is done in function `color_gradient_threshold` in 8th cell of the Ipython Notebook. 

The transformation process with the origin and destination images is shown in the following image, which it can be seen that the lines appear parallel in the warped image:
![alt text][image4]

And this image show the transformation applied in an image with curved lines:
![alt text][image5]

#### 4. Find lines.

Once the image is thresholded and transformed to birds eye view, an histogram of nonzero pixel is used to detect the line base (at the bottom of the image) and then the method of the sliding window is used to find the pixels that form the lines. Then this pixels are interpolated using a 2 degree polynomial that is taken as the line approximation.

This process is done in function `find_lines` in 12th cell of the Ipython Notebook. Example of line detection:
![alt text][image7]

Once lines are detected this information can be used to calculate the lines in the following image looking for the pixels in the proximity of the estimated line. This pixel search is simpler than that of the sliding window and reduce computing time. If the pixel found are below a certain threshold the sliding window search is performed.

This process is done in function `find_lines_assisted` in 14th cell of the Ipython Notebook. Example of assisted line detection:
![alt text][image8]

#### 5. Radius of curvature of the lane and the position of the vehicle with respect to center.

These calculation are done in functions `calc_curvature` and `calc_position` in 16th and 17th cells of the Ipython Notebook.

To make the calculation of the curvature an estimation of a 30 m long and 3.7 m wide is used. In the case of the position the 3.7 m of the lane wide is compared to distance between lines in the intersection of the line previously detected with the bottom of the image to estimate the pixel to meters conversion.


#### 6. Example image of line detection plotted back down onto the road.

Finally that part of the image between the left and right lines will be the lane and it is colored in the original image. This is done in function `print_lane` in 18th cell of the Ipython Notebook.

Firstly the space between lines in the warped image is colored, then this image is un-warped using the inverse of the M matrix and finally the line is super-imposed in the undistorted original image.The following image is an example of the result on a test image (position and curvature also printed):
![alt text][image9]

---

### Pipeline (video)

This image pipeline transformation is applied to the videostream through the function `full_image_pipeline` in 22th cell of the Ipython Notebook.

To allow for the use of information from previous images in the image being processed a helper class `Lines` is used (defined in 21th cell of the Ipython Notebook). This class allows for averaging the lines of last n images (in this case n = 3) and to calculate the position and curvature every three images instead of in every image that will change data too fast and would be uncomfortable.

#### Video output. 

Here's a [link to video result](./result.mp4)

---

### Discussion

#### 1. Issues faced in the implementation of the project.

There are two main issues to address in the implementation of the project:
First one is the thresholding function: finding the appropriate color spaces, channels, and threshold is more an art than a science and no combination was found to be robust enough for challenge videos. Further improvement is possible in this function.

The other one is the use of previous lines to help in the actual line detection and made it more robust and stable. And also, related to line finding and approximation, the use of higher order polynomial to fit the lines for curvy roads.