# **Advanced Lane Finding Project**
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


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

[image1]: ./output_images/chessboard_lines.png "ChessBoard"
[image2]: ./output_images/undist.png "Road Transformed"
[image3]: ./output_images/gradient_filters.png "Binary Examples"
[image4]: ./output_images/combined.png "combined Example"
[image5]: ./output_images/birdeye.png "Fit Visual"
[image6]: ./output_images/histogram.png "histogram"
[image7]: ./output_images/detecting_lane_lines.png "lane-lines-highlighted"
[image8]: ./output_images/output.png "Output"
[image9]: ./output_images/radius.png "radius"

[video1]: ./output_videos/project_video_processed2.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

## Steps of Lane Finding

### Camera Calibration

The code for this step is contained in the first code cell of the IPython notebook located in "advanced_lane_finding.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Calibration][image1]

### Pipeline (single images)

#### 1. Undistorting Image

After calibrating the camera and extracting the camera matrix and distortion coefficients these values will be the input for cv2.undistort() to remove any distortion.
![Before & After undistortion][image2]

#### 2. Appling gradient filter and color transform (and identify where in your code)

First I tested each sobel operator and tried to obtain good thresholds for each. Also I converted the image to hls and used the s channel to be able to filter out the eniroment and leave the lane lines clear. 

![Gradients][image3]

By combining the gradients it helped completeing the lane lines that was not clear using one filter only. 
The functions are in the 3rd code cell of the IPython notebook and the combined binary is in the 5th cell.

![Combining][image4]


#### 3. Bird's Eye (Perspective Transform)
To be able to detect the lane curvature the image must be as if it was taken from top view or as it is called bird's eye view. 
The code for my perspective transform includes a function called `warper()`, which appears in the 6th code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([(150,680), (550, 460), (750, 460), (1200,680)])
dst = np.float32([(200,680),(200,0),(1100,0),(1100,680)])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 150, 680      | 200, 680      | 
| 550, 460      | 200, 0        |
| 750, 460      | 1100, 0       |
| 1200, 680     | 1100, 680     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.


![Perspective Transormation Verification][image5]

#### 4. Lane Lines & Sliding Window 
After Transforming the image and applying the gradient, I plotted a histogram to locate the left and right lane line. This is done by detecting the peaks of the historgram. Cell Code Block 7.
![Histogram Graph][image6]
I used the two highest peaks from the histogram as a starting point to know where the lane lines are and thhen applying the sliding windows which moves along the road to locate where the lane lines go. To be more efficient in the video I used the previous lane line position as a start for the following frame instead of doing a blind search each frame. Cell Code Block 8.
![Sliding windows][image7]

#### 5. Curvature of the lane and the position of the vehicle.
From the second order polynomial that were used to locate lane line pixels
The Radius of Curvature equation is:
![Radius of Curvature equation][image9]

The values must be convert from pixels to real-world meter measurements. Cell Code Block 10.

#### 6. Overlaying the lane area 

Now I have to overlay the results in the video to show clearly the lane are and the vehicle position so I took the warped image and the x&y points to fill the lane with color. Then I will use the warp function used for perspective transform but I will invert the dst and src numbers to lay down the colored lane are in position.
I used cv2.puttext to add the lane curvature and vehicle offset on the top of the video.
Cell Code Block 11.
![Final Image][image8]

---

### Pipeline (video)

#### 1. Project Video

When I tested the pipline on the video specificly in the duration 39 to 22 sec the output was not satisfing as the overlayed area started to misplaced. I tried a couple of combinations and thresholds however it failed to fix this duration. I search for other color spaces to test and try to make it more robust and I have found the "LAB" 
* L – Lightness ( Intensity ).
* a – color component ranging from Green to Magenta.
* b – color component ranging from Blue to Yellow.
Cell Code Block 26-27.
After finding a sxuitable threshold I added it to the image process and this was my final result.
Here's a [link to my video result](https://youtu.be/B5Lr3F7Bj2g)

---

### Discussion

#### 1. Challenges and Improvements

I tried the challenge video and the image process used in the project video failed so I adding the l channel from hls and it fixed most of the video however, it still need more work. As for the harder challenge video the process dealt with alot of complex condition as the look up region was too long than the lanes, the curves was more sharper, reflection of the vehicle interior appeared in direct sunshine and the cars from opposite lane. 

To improve the process 
- search for optimum thresholds and binary images combinations must be done.
- Decrease the look up region of the lane.
