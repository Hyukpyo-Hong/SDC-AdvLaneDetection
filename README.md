
**Advanced Lane Finding Project**

The goals/steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to the center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undistort.jpg "Undistorted"
[image2]: ./output_images/test_undistort2.jpg "Road Transformed"
[image3]: ./output_images/test_binary2.jpg "Binary Example"
[image4]: ./output_images/test_linedrawing.jpg "Warp Example"
[image5]: ./output_images/test_lane_detect.jpg "Fit Visual"
[image6]: ./output_images/test_final2.jpg "Output"
[video1]: ./project_video_lane.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `calibration` and `undistort` methods of "./P4_Answer.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image with `binary()' function. Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform()`, which appears in  in the code cell of the IPython notebook.  The `transform()` function takes as inputs an image (`img`), and uses source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32([[592,450],
                  [260,680],
                  [1057,680],
                  [691,450]])
dst = np.float32([[(img_size[0] / 4), 0],
                  [(img_size[0] / 4), img_size[1]],
                  [(img_size[0] * 3 / 4), img_size[1]],
                  [(img_size[0] * 3 / 4), 0]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 592, 450      | 320, 0        | 
| 260, 680      | 320, 720      |
| 1057, 680     | 960, 720      |
| 691, 450      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like below,
And this code is implemented in `findLane()` function of notebook file. To make more smooth lane identification, I stored detected x-values and their fit of previous 15 frames, then calculate average values of them. Also, I used `sanityCheck()` function to make sure my detection is correct. If sanity check is failed, then I removed current detected information from the stored array. 

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `measure_curvature_offset()` function of notebook file. To compute curvature, I used curvature formula, and to convert pixels to real world plots, I assumed the ratio, 30 meters per 720 pixels on the y-axis, and 3.7 meters per 700 pixel on the x-axis. Then to calculate the position of the vehicle from the center, I subtracted the mean x-values of the bottom of the left and the right lane from the middle point of the x-axis, then multiplied the ratio of the x-axis.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 258 through 321 in my code in `findLane()` function of my notebook file. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_lane.mp4)

---

### Discussion

#### 1. Briefly discuss any problems/issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

#### (1). Figuring out optimized values for each threshold

I used jupyter notebook `widgets` from `IPython.html` to make easier to find good values for lane detection. I combined `Sobel operator`, `Magnitude of the Gradient`, `Direction of the Gradient`, and `L channels` of HSL mode. It works great when I applied uniformed images, but not works on challenge video with many obstacles such as the sunshine, sharp curve, shadows and so on. So my `binary()` function is not suited well for these images. If I have more time, I will prepare images of this kind of situation. And I will test with my widget to find good values, or I will find another approach to solve this problem.

#### (2). Curvature of straight lane

On straight lane images, the curvature of the left lane, and right lane shows great difference each other, since the curvature of the straight lane is huge and sensitive to the small difference. So my `sanityCheck()` often check error, when the curvature of left lane is 50,000m, and the right lane is 20,000m, for example. Although both of them indicate almost straight line, their big difference seems like an error to my `sanityCheck()`. So, I decided if one of the lane's curvature is more than 3Km, I ignored their difference. The result is good on my project video, but I'd better think about good sanity check for two lanes similarity.

