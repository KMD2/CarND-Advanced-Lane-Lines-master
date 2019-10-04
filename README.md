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

[image1]: ./camera_cal/calibration1.jpg "Chess_orig"
[image2]: ./output_images/test_undist_calibration1.jpg "Chess_undist"

[image3]: ./output_images/Undistorted_test1.jpg "Undistorted"
[image4]: ./output_images/Thresholded_test1.jpg "Threshold Example"
[image5]: ./output_images/Warped_test1.jpg "Warp Example"
[image6]: ./output_images/Detected_test1.jpg "Fit Example"
[image7]: ./output_images/Final_test1.jpg "Final Example"

[video1]: ./output_video/project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the cells under "Camera Calibration" under step 1 and step 2 at the top of the IPython notebook located in `./Advanced_Lane_Finding.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

Original image             |  Undistorted image
:-------------------------:|:-------------------------:
![alt text][image1]       |  ![alt text][image2]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one (test1):
![alt text][image3]

I used the `undistort()` function which applies the `cv2.undistort()` that takes the camera matrix and distortion coefficients from the ones found in the camera calibration step.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps under "Pipeline (test images)" --> "Create a thresholded binary image using color transforms and gradients"  in `Advanced_Lane_Finding.ipynb`).  I used Sobel operators (x and y direction), gradient magnitude, and the direction of the gradient thresholds with values (10,150), (100,255), and (0.7,1.4) respectively. I also applied color thresholds on the S channel form the HLS color space and the L channel from the LAB color space with threshold values of (100,255) and (150,255) respectively. I ended up using a combination of these thresholds to obtain a binary image. below is an example of my output for this step.  
![alt text][image4]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform (under Pipeline (test images) --> Perspective transform ).  The `warp_image()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner starting from the top right corner and ending in the lower right corner for the source points and starting from the lower right corner and ending up in the top right corner for the distinction points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 678, 443      | 919, 0        | 
| 605, 443      | 285, 0      |
| 285, 665      | 285, 665      |
| 1019, 665     | 919, 665      |

Below is an example of a warped undistorted thresholded image (test1):

Undistorted image          |  Warped image
:-------------------------:|:-------------------------:
![alt text][image3]       |  ![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Under Pipeline (test images) --> Identify lane-line pixels , I used the histogram/sliding windows method implemented in   `find_lane_pixels()` function. Then I used the function `fit_polynomial()` to fit the lane lines pixels (2nd order polynomial). I also used the search from prior method implemented in `search_around_poly()` function; to skip using the sliding window method (blind searching) in future frames. Below is an example of detecting the lines in image test1:

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this under Pipeline (test images) --> Measuring Curvature  in `Advanced_Lane_Finding.ipynb` using `measure_curvature_real()` function.  The function calculates the left and right lines' curveture using the radius of curveture equation and then returnes the average of the two. The lines of code below showes how i calculated the shift (variation) of the vehical's center from the center of the lane:

```
left_curverad = ((1+(2*left_fit_cr[0]*y_eval*ym_per_pix+left_fit_cr[1])**2)**(3/2))/np.absolute(2*left_fit_cr[0])  ## Implement the calculation of the left line here
right_curverad = ((1+(2*right_fit_cr[0]*y_eval*ym_per_pix+right_fit_cr[1])**2)**(3/2))/np.absolute(2*right_fit_cr[0])  ## Implement the calculation of the right line here
avg_curverad = (left_curverad + right_curverad)/2

lane_width = 1019 - 285 - 100 
vehicle_center = (1240/2 - 285)/(1019-285) * (lane_width) + 285
center_left_lane_cor = left_fit[0]*h**2 + left_fit[1]*h + left_fit[2]
center_right_lane_cor = right_fit[0]*h**2 + right_fit[1]*h + right_fit[2]
lane_center = (center_left_lane_cor + center_right_lane_cor)/2
variation = (vehicle_center - lane_center) * xm_per_pix
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step under Pipeline (test images) in the function `lane_projection()`.  Here is an example of my result on  test1 image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, to avoid failure in the video lane detection. In the pipeline, for the first frame I used the histogram method to detect the lane line pixels, then I performed some sanity checks to make sure the detection was performed correctly. For the following frames, I checked whether the previous frame was detected correctly, if so, I used the searching from prior method to detect the lane line pixels, then I performed the sanity checks like before. The sanity checks can be found under Video Pipeline --> in the `Line()` class and `video_pipeline()` function.

The video pipeline works fine for the current test video, but I believe it will fail in a video with a different lighting/road conditions, as the thresholds is my pipeline are overfitted to the video I tested. So the thresholds should be properly adjusted for the pipeline to succeed in different conditions.

Another noticeable issue in the video is that some times lane detection is a bit jumpy, and that is because when I performed the perspective transform I assumed that the road is flat unlike in some parts of the video.
