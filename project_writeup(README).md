## Writeup Template


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

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./P2.ipynb" in a function called 'calibrate_camera'

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. Most of the example code for this can found in the course.

I then used the output `objpoints` and `imgpoints`(corners) to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image in the third cell of notebook in 'pipeline' function. For this I used Sobel operator and hue layer of the image with similar threshold as used in the course. Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective()`, which appears in the 4th code cell of the IPython notebook).  The function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
    offset = 100
    # image.shape = (720, 1280)
    src = np.float32([[538, 489], [747, 489],  
                             [1040, 677], 
                             [265 , 677]])
    dst = np.float32([[image.shape[1]//4, offset], [3*image.shape[1]//4, offset], 
                             [3*image.shape[1]//4, image.shape[0] - offset], 
                             [image.shape[1]//4,  image.shape[0] - offset]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 538, 489      | 180, 100      | 
| 747, 489      | 540, 100      |
| 1040, 677     | 540, 620      |
| 265 , 677     | 180, 620      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I use the function `find_lane_pixels_plot_base_curve()` and `fit_polynomial()` which take a binary warped image after perspective transform and draw sliding windows on it to finally find the x coordinated of the lane lines to be drawn. This works on the initial frame when I dont have any previous lane lines drawn with me.

Once I have values for the first lane lines(both left and right lane), I use functions `fit_poly()` and `search_around_poly()`
which drawn lanes lines based on the previously drawn lane lines(as per the second method explained in the sliding window section of the course. Most of the code is taken from example provided in the course.

One major change which I did was to change the 'margin' paramter from 100 to 50 as there was too much noise in some of the images and my lanes were not being drawn accurately, using a smaller margin helped in correcting this issue

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I have a function `poly_rl_coeff()` which is called by the functions from the previous section to calculate the radius of curvature of the lanes, I also return col(Centre of lane) from those function and use it calculate relative position of the vehicle with respect to centre of the lane.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In the cells 13 and 14, I define a process which call the above functions and plots the detected lanes on the image

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Perfonally, I feel I could have written code in a cleaner way to avoid some of the troubles as at one points there were too many function and variables calling each other that I lost track of which variable and function signifies what. Initially I had faced an issue that one of plotted lanes were curving off from the original track, due to noise in the image. This was corrected using a smalled margin. 
From the challenge video I found that sometimes, some other lines other than the lane lines could have more pixels in the histogram and hence might get detected as the original lane.
Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
