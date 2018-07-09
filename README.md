# Advanced-Lane-Finder

![alt text][image20]

---

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

[image1]: ./examples/calibration-distorted.png "Calibration Distorted"
[image2]: ./examples/calibration-undistorted.png "Calibration Undistorted"
[image3]: ./examples/original.png "Original Road"
[image4]: ./examples/undistorted.png "Undistorted Road"

[image5]: ./examples/blue-channel.png "Blue Channel Threshold"
[image6]: ./examples/lightness-channel.png "Lightness Channel Threshold"
[image7]: ./examples/combined-binary.png "Combined Binary"
[image8]: ./examples/persp-boundary.png "Perspective Boundary"
[image9]: ./examples/binary-warped.png "Binary Warped"
[image10]: ./examples/histogram.png "Histogram"
[image11]: ./examples/sliding-window-fit.png "Sliding Window Fit"
[image12]: ./examples/targeted-fit.png "Targeted Fit"
[image13]: ./examples/output.png "Output"


[image17]: ./examples/r-curve-eqn.png "Curve EQN"
[image18]: ./examples/curvature.jpg "Measure Curvature"

[image19]: ./examples/lane-overlay.png "Lane Overlay"
[image20]: ./project-animation.gif "Animation"

[video]: ./project_video.mp4 "Project Video"


### Camera Calibration

The code for this step is contained in the first code cell of the IPython notebook located in `./Advanced-Lane-Finder.ipynb`.  

I start by preparing "object points", which will be the `(x, y, z)` coordinates of the chessboard corners in the world.
Here I am assuming the chessboard is fixed on the `(x, y)` plane at `z = 0`, such that the object points are the same for each calibration image.
Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect 
all chessboard corners in a test image.  `imgpoints` will be appended with the `(x, y)` pixel position of each of the corners in the image plane
with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()`
function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]
![alt text][image2]

### Pipeline (single images)

#### 1. Distortion-corrected Example

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]
![alt text][image4]


#### 2. Color Threshold Samples

I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step. 

The lightness channel worked well for white lines but not yellow linee and blue channel worked very well for picking up the yellow lines.

I filtered the image using these two channels and then combined them to get the final binary image.


![alt text][image5]
![alt text][image6]
![alt text][image7]

#### 3. Perspective Transform Samples

The code for my perspective transform includes a function called `persp_transform()`.  The function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points so I could get the most accurate transformation:

```python

offset = 400
w = img.shape[1]
h = img.shape[0]

src = np.float32([
    (575, 464),
    (710, 464), 
    (220, 710), 
    (1088, 710)
])

dst = np.float32([
    (offset, 0),
    (w - offset, 0),
    (offset, h),
    (w - offset, h)
])

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 464      | 400, 0        | 
| 710, 464      | 880, 0        |
| 220, 710      | 400, 720      |
| 1088, 710     | 880, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image8]
![alt text][image9]

#### 4. Identifying Lane Lines

Then I used a sliding window search to find lane line pixels within a margin around the histogram peaks.
Then a 2nd order polyfit is used to find equations of the lane lines. The pipeline keeps track of the last `n` line fits and updates it's best fit property depending on the percent difference of each estimation.

![alt text][image10]
![alt text][image11]
![alt text][image12]

#### 5. Calculating Curvature and Offset

I calculate curvature and offset in `calculate_curve_offset()`. Example output:

![alt text][image17]
![alt text][image18]


#### 6. Plotting Lane Line Overlay

I implemented this step in the function `draw_lane()`.  Here is an example of my result on a test image:

![alt text][image13]

---

### Pipeline (video)

Here's a [link to the project video output](./output_videos/project_video.mp4)

---

### Discussion

I started with straightforward steps of camera calibration, undistorting and perspective transform. The gradient and color thresholds took some trial and error. I found that the gradient in the x-direction worked best on the lightness channel. I also combined two color thresholds: blue and lightness channels. The blue channel worked very well at extracting the yellow lines and the lightness channel picked up white lane lines (and occasional bright wheel rims).

The pipeline worked well on the project video. I tried the same pipeline on the harder challenge videos but they were total failures. The color channel thresholds were capturing too much peripheral noise from shadows, cars and scenery, resulting in very poor lane line approximations.

Some tweaking of the color thresholds would help but it would be ideal if cropping could be used to completely eliminate peripheral objects. Perhaps a tighter perspective transform but of course this would make assumptions about the lane widths and road conditions.

Perhaps using a method similar to the sliding window search but applied to the color thresholds could help. Knowing that as the region moves up the image toward the horizon, more noise is expected and color thresholds can be adjusted.
