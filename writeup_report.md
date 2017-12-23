## Project Writeup

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

[image1]: ./output_images/undistorted_img.png "Undistorted"
[image2]: ./output_images/resized_img.png  "Road Transformed"
[image3]: ./output_images/binaryImage.png "Binary Example"
[image3a]: ./output_images/stackedPlotSubtract.png "Stacked Plot"
[image4]: ./output_images/birdsEye.png "Warp Example"
[image5]: ./output_images/windowedFit.png "Fit Visual"
[image6]: ./output_images/processedImage.png "Output"
[video1]: ./output_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.   

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the IPython notebook located in "./AdvLaneLines.ipynb" under the title "Calculation of the Distortion Correction Parameters"  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

For data encapsulation purposes, I load the camera matrix, distortion coefficients, rotation and translation vectors into a custom class `CameraParams` for storage and retrieval. This allows me to recall the saved parameters after running the calibration once. The parameters are saved in a pickled structure named `camParams.p`.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

With the distortion correction parameters found and tested, I applied them to the sample test images to ensure the work on non-checkerboard images. An example is shown below. Notice how the reflection from the dash is curved in the left (distorted) image and straight in the right (undistorted) image. To achieve this, I simply call the cv2.undistort function with the camera matrix and distortion coefficients restored from the pickled data. The concatenated images are here:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. I encapsulate these steps in the function `thresh_and_binary()`. I used several color spaces in order to maximize the visibility of the lines under various lighting conditions. The V channel from HSV provides the most consistent white and yellow line visibility moderate to high contrast images (darker road than the line), S channel from HSV is reasonably good for detecting yellow lines under various conditions. The S Channel from HLS is used to filter broad bright areas in order to obtain better contrast of the lines. The output of my steps applied to one of the test images is seen here:

![alt text][image3]

For debug purposes, I created a `test_thresh()` function that outputs three different layers of my choosing for use in a stacked plot in order to enhance/debug/develop my thresholding and gradient function. One layer would be the detected shadows of an image to be removed (in red) and another layer would be the existing thresholding output (in green). Where there is overlap (deleting too much data in the case of a removal mask) will show up as the combination, in yellow. An example of a stacked plot is shown here:

![alt text][image3a]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which is below the `Perspective Transform` heading in the file `AdvLaneLines.ipynb`.  The `warp()` function takes as inputs an image (`img`), the transformation matrix (`tMtx`) and a flag whether this is a forward or inverse transformation (`inverse`). Because the transformations I perform are from the same perspective and back to the original, I chose to implement my function in this manner. The transformation matrix for warping (`M`) and unwarping (`M_inv`) are pre-calculated, stored, and passed into the `warp()` function as required. The transformation points are hardcoded as follows:

```python
src = np.float32([[(200,720),(580,480),(720,480),(1050,700)]])
dst = np.float32([[(290,720),(400,190),(930,190),(960,720)]])
```

I verified that my perspective transform was working as expected by testing on the straight lines test images and inspecting its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I wrote a helper function `fitLanes()` in my notebook `AdvLaneLines.ipynb` to compute the left and right second order polynomial parameters, the y-values used to plot these functions, and the respective left and right x values used. This function takes an optional visualization parameter for use within the notebook to output a debug display as seen below:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate the curvature of the lane, I used the `Rcurve()` function in `AdvLaneLines.ipynb`. I first rescale the x and y values used in the polynomial fit  with pixel to metre values provided in the course. Then, I re-perform the fit to obtain the second order parameters, which are passed to Rcurve to compute the radius, in metres. I repeat the process respectively for left and right lanes.

For calculating the position of the vehicle relative to the centre of the lane, I wrote the function `distFromCentre()`, which takes in the pixel-valued polynomial fit parameters for both left and right fit lanes and the scale value for converting from pixels to metres in the x direction. The result is a measurement of the lane centre's position right (positive) or left (negative) relative to the car's centre (assumed to be the image centre.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Within my notebook under the heading `Outputting the Results`, my `pipeline()` function maps the detect lanes onto the original image.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In order gain better intuition on the limitations of my implementation, I ran my algorithm against the challenge video and obtained less than stellar results. Where the model fails is on heavily repaired roads with crack repairs that are in the direction of travel (vertical as viewed by the camera). The algorithm mistakes these repairs as lane lines and therefore incorrectly interprets the lane lines.

Where I could improve the pipeline is to  calculate secondary and tertiary potential lane lines, take into account details about the features I calculate, like distance between detected lines, and reject ill-fitted primary lines and opt for secondary detected lines if these are a better fit.

A [link to my challenge video output](./challenge_output_video.mp4) is included with this submission. This output includes intermediate binary output images for debugging what the pipeline detected through image processing.
