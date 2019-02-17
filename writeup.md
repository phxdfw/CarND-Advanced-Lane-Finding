## Writeup on Advanced Lane Finding

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

You're reading it! LOL

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first a few code cells of the IPython notebook located in "`./project2.ipynb`".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Calibration Result][./output_images/camera_cal_test1.jpg]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Before][./test_images/test1.jpg]

Here is the result after distortion correction:
![After][./output_images/undistort_test.jpg]

For the distortion correction, the parameters `matrix` and `distortion` generated from the camera calibration step is used to feed method `cv2.undistort`, and then I can see some slight difference between the images before and after distortion correction (mainly on the borders and corners where there are usually more distortion by lens).

There are much more images available in my IPython notebook inline.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at the following "Thresholding" section right after distortion correction).  Here's an example of my output for this step.  

![After applying threshold][./output_images/threshold_test.jpg]

I used HLS color transform and got `L` and `S` channels. `S` channel and Sobel gradient on `L` channel are used together for the combined binary from thresholding.

For more images, please take a look at the inline output images in my IPython notebook.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in the next section called "Perspective transformation" in the file [`project2.ipynb`](./project2.ipynb).  The `warp()` function takes as preprocessed inputs an image (`undistort_gray_image`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
yoffset = 100
xoffset = 270

src = np.float32([
    [510, 500],
    [780, 500],
    [1120, 700],
    [200, 700],
])
dst = np.float32([
    [xoffset, yoffset],
    [img_size[0] - xoffset, yoffset],
    [img_size[0] - xoffset, img_size[1]],
    [xoffset, img_size[1]],
])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 510, 500      | 270, 100      | 
| 780, 500      | 1010, 100     |
| 1120, 700     | 1010, 720     |
| 200, 700      | 270, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image, which could be seen in the inline image printing in my IPython notebook.

![wrap verification][./output_images/warp_test.jpg]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff in the following section "Sliding Window Search and Polynomial Fitting", and fit my lane lines with a 2nd order polynomial kinda like this:

![polynomial fitting][./output_images/poly_test.png]

The idea is to use sliding windows to track left and right lanes separately. Also, I really like the tool `np.polyfit` which comes very handy.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the following block named "Measure Curvature" in my code in `project2.ipynb`. The basic idea is to use the instruction and formulas presented in course. Variables `ym_per_pix` and `xm_per_pix` are used for conversion between pixel and meter.

I directly copied and pasted some codes from the course material and quiz, as the algorithm is rather self-explanary. It would be perfect if cv2 had some lib on this function to make it less messy!

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the next section named "Visual display of the Lane Boundaries and Texts" in my code in `project2.ipynb` in the function `draw()` and `write_data()`.  Here is an example of my result on a test image:

![Final Example][./output_images/test6.jpg]

More examples of this type are available from the inline display on my IPython notebook.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result on project_video](./project_video_out.mp4)
Here's a [link to my video result on challenge_video](./challenge_video_out.mp4)
Here's a [link to my video result on harder_challenge_video](./harder_challenge_video_out.mp4)
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In finding the lines, my method of searching from prior does not work quite well. It seems to be working in my unit test right after the method body, however, somehow it fails when working on the video. TO generate the video results, I commented it so the video was using the basic sliding window approach. I will need more time to debug this.

From the video result, it worked basically in `project_video.mp4` and `harder_challenge_video.mp4`, however, for `challenge_video.mp4` it got trapped by the wrong lines. So there is likely an assumption that the lanes on the road are clear and the car is driving not accrossing the lanes. It is good that my work is not quite confused by the double yellow lines in `harder_challenge_video.mp4`. I believe I could tune the thresholding better for better results in `challenge_video.mp4`, if I had more time working on this.

Also, through working on this project, I feel it is really a good habit to have a unit test case after each functionality I developed in the pipeline. It encourages myself a lot on looking at the effects on the inline image, also it eases myself a lot in debugging and writeup.