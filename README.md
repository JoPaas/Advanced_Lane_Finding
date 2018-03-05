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

[image1]: https://github.com/Nervehurter/Advanced_Lane_Finding/blob/master/output_images/calibration.JPG "Undistorted"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook "P4.ipynb". I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.

Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)
[//]: # (Image References)

[image2]: https://github.com/Nervehurter/Advanced_Lane_Finding/blob/master/output_images/undistort.JPG "Road Transformed"
[image3]: https://github.com/Nervehurter/Advanced_Lane_Finding/blob/master/output_images/binaryMask.JPG "Binary Example"
[image4]: https://github.com/Nervehurter/Advanced_Lane_Finding/blob/master/output_images/warped_straight_lines.JPG "Warp Example"
[image5]: https://github.com/Nervehurter/Advanced_Lane_Finding/blob/master/output_images/hist.JPG "Histogram"
[image6]: https://github.com/Nervehurter/Advanced_Lane_Finding/blob/master/output_images/polyfit1.JPG "Fit Visual 1"
[image7]: https://github.com/Nervehurter/Advanced_Lane_Finding/blob/master/output_images/polyfit2.JPG "Fit Visual 2"
[image8]: https://github.com/Nervehurter/Advanced_Lane_Finding/blob/master/output_images/output.JPG "Output"

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. I defined functions for different types of thresholds. I the end I used a color threshold on the HLS 'S' channel (100 - 255) together with directional sobel (0.7 - 1.3) and absolute sobel (40 - 100). Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `birdview_transform()`, which appears in the 5th code cell of the IPython notebook.  The `birdview_transform()` function takes as inputs an image (`img`), as well as perspective transform (`M`).  I chose to hardcode the source and destination points in the following manner in the `get_perspective_transform()` function:

```python
    # source points adjacent to lane lines
    src = np.float32([[700, 450],
                [1130, 660],
                [585, 450],
                [200, 660]])
    # target points on warped image
    dst = np.float32([[img_shape[0]-pad_r, pad_top],
                      [img_shape[0]-pad_r, img_shape[1]-pad_bottom],
                      [pad_l, pad_top],
                      [pad_l, img_shape[1]-pad_bottom]])
```


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then to initialize the two lanes I used a histogram (blue) of nonzero pixels in the bottom half of the image. this can be seen in the image below.

![alt text][image5]

Next I used a sliding window search with `margin = 100`. To get a robust initialization, at least 4 of the 9 windows must detect positive pixels in the binary image. this detection is shown in the next image.

![alt text][image6]

When the lines are initialized the function `find_lane_polinomials()` looks for positive pixels in the binary image with a margin of 100 pixels around the last detection. the searchig area is depicted in the next image.

![alt text][image7]

The parameters of the fitted polynoials are stored in the `Line()` class and filtered over the last 5 iterations using a ring buffer (imported from external source) for more stability.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the function `get_curv()`. It takes the polyline pixels and stretches the pixels into real coordinates (it is only an estimate). Then the curvature can be calculated using the formula from lesson 15 slide 35 "Measuring Curvature". Curvature is then also stored in a ring buffer per line and a mean is calculated for the lane with the `update_curv()` function in the `Line()` class.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `draw_lane()` in the 5th code cell. the `process_image()` function then adds the values as text to the image. Here is an example of my result on a test image:

![alt text][image8]

---

 
### Pipeline (video)

[//]: # (Image References)

[video1]: https://github.com/Nervehurter/Advanced_Lane_Finding/blob/master/output_videos/project_video_3.mp4 "Video"

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://github.com/Nervehurter/Advanced_Lane_Finding/blob/master/output_videos/project_video_3.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

It took some time to adjust the thresholds to generate the binary image. Here a lot of performace can be gained/lost because the window search and other techniques work better on better preprocessed images. Although the filtering is necessary to stabilize the outputs, it introduces a delay in the lane output. Here an observer like a kalman filter could be used to minimize the effects. In general to improve the performance I would like to estimate the pitch of the vehicle, because this influences the points for warping and thus the line shapes. Also a technique to cope with multiple lines per side could be benificial, since in city scenarios there are often multiple lanes.