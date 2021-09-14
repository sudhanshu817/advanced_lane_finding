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

[image1]: ./output_images/undistorted.jpg "Undistorted"
[image2]: ./output_images/undistorted_road.jpg "Road Transformed"
[image3]: ./output_images/thresholded_image.jpg "Binary Example"
[image4]: ./output_images/warped_img.jpg "Warp Example"
[image5]: ./output_images/polynomial_fit.jpg "Fit Visual"
[image6]: ./output_images/output.jpg "Output"
[video1]: ./project_video_output_combined.mp4 "Video"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

The actual pipeline is present in advanced_lane_finding_pipeline.ipynb, draft.ipynb is the initial draft which has all the methods implemented and intermediate results were saved using this notebook.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell 2,3 and 4 of the IPython notebook located in "./advanced_lane_finding_pipeline".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images ('test_images/test1.jpg') to get output like this one:
![alt text][image2]

After getting the camera matrix and distortion coefficients, I used cv2.undistort using the camera matrix and distortion coefficients to get distortion correction image.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at cell 6 in advanced_lane_finding.ipynb).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

The final thresholded image was generated using this logic ((dir_binary == 1) &(grady==1) &(mag_binary==1)) |(hls_binary==1).

Where dir_binary is the thresholded image using gradient direction with a sobel filter,
grady is the thresholded image using gradient in y direction with a sobel filter,
mag_binary is the thresholded image using magnitude of gradient,
hls_binary is the thresholded image using the 's' channel of hls format of the image.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `get_warp_matrix()`, which appears in cell 7 of the ipython notebook. The function takes as inputs an undistorted image. The src and dst points are defined in the same function.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[img_size[0]//7+20,img_size[1]],#img_size[0]//7, img_size[1]],
                      [(6*img_size[0])//7+30, img_size[1]],
                      [img_size[0]//2+60, img_size[1]//2+100],
                      [img_size[0]//2-60, img_size[1]//2+100]])
offset=200
dst = np.float32([[offset, img_size[1]], 
                     [img_size[0]-offset, img_size[1]],
                     [img_size[0]-offset,0],
                     [offset, 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 202, 720      | 200, 720      | 
| 1127, 720     | 1080, 720     |
| 700, 460      | 1080, 0       |
| 580, 460      | 200, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for identifying the lane-line pixels and fitting a polynomial is present in cell 12 and 13 of the advanced_lane_finding.ipynb notebook.
For the first frame we find the histogram peaks in the warped image and then use a moving window up the frame of image to find the lane-pixels. After getting all the points, I fit a 2 degree polynomial on thsoe points using np.polyfit.
For the sub-sequent frames, I only search around the results of the previous frames.
The output after fitting lines (for test_images/test6.jpg) on the warped image looks like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is measured in cell 14 of the notebook and the vehicle distance from centre is calculated in cell 16. Considering the camera is mounted in the centre at the top of car, the distance of lane centre from the centre of image is the vehicle distance from the centre, which is calculated by finding the midpoint of the two fitted polynomials at the base of the image, and then calculating it's distance from the centre of the image. This is the distance in pixels, multiply it by xm_per_pix to get the actual distance in KM.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This code is implemented in cell 15 of the ipython notebook, it plots both the polynomial on the image and then draws them on the warped image using cv2.fill poly. It also takes the inverse perspective transform matrix to get the original image from the perspective transformed image.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output_combined.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Getting the binary thresholded image using various combinations was very difficult. The first implementation of the binary thresholding part resulted in some continous white lines present on the road also being selected by the filter. As a result these white lines where being detected as the lane lines and the polynomial equations got fit on these lines resulting in wrong markings and this was hard to debug.
The pipeline fails in regions where the road is highly illuminated with sunlight resulting in wrong estimation of the lane lines which are white.
To make the pipeline more robust we can improve the binary thresholding function by giving more weight to the hls binary filter as the 's' channel of the hls image is more robust to different lightings and illumination on the road.
