# Self Driving Car Nanodegree
### Project 3 - Advanced Lane Detection
---

**The goals/steps for this project are the following:**
1. Distortion correction using images of chessboards.
2. Perspective transformation to view lane lines from straight above.
3. Processing images to highlight lanelines.
4. Sliding window search to detect lane lines and their curvatures.
5. Drawing the detected lane lines on the original images.

Complete codes and results for this project can be found in the  [`P3-shorter.ipynb`](https://github.com/kwonjh90/SDCND-AdvancedLaneFinding/blob/master/P3-shorter.ipynb)

### 1. Unidstorting an image.
Images taken with cameras have distortions, which occur while putting 3-D views into a 2-D planes(images). These distortions can be recovered once relevant parameters are determined. Such parameters include a camera matrix and a distortion coefficient.

In the cell below **1.1**, I started by preparing "object points", which are the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, `objp` is just a replicated array of coordinates, and `objpoints`will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `mgpoints`to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the test image using the `cv2.undistort()` function.
Below are the results:  

![chessboard_undistort][./report_images/q1_1.png]

### 2. Color transformation and gradient thresholding.
We need to further process the undistorted images to clearly distinguish and detect lane lines. Two techniques used are color transformation and gradient thresholding.

As can be seen in the cell below section **3.5**, a combination of thresholdings where applied to an image to effectively extract lane lines:  
* Thresholding a L-Channel -> Sobel X
* Thresholding a S-Channel

The resulting image is shown below:  
![transformation_binary][./report_images/q2_1.png]

### 3. Perspective Transformation.
Images taken from a front-facing camera on a vehicle have another distortion due to perspective; objects closer to the camera look bigger and that further away smaller. This distortion due to perspective make lane lines seem to converge together, even though they are in fact nearly parallel lines.  
For this reason, we need to transform the images to be viewed from the "birds-eye view". This view will allow the curvatures of lane lines to be displayed and measured accurately. I implemented an OpenCV `cv2.warpPerspective()` method to achieve the perspective transformation.  
In the cell below **2.1**, I loaded a clean image to manually select four points that aligns near-perfectly with lane lines. Then, in the cell below  **2.2**, I used the four points to warp the image.

The resulting images are shown below:  
![before_warp][./report_images/q3_1.png]  
![after_warp][./report_images/q3_2.png]  

### 4. Lane Line Detection
Now that we have images that is warped to "birds-eye views" and pixels of lane lines highlighted with image processing, we can do the actual detection based on a number of assumptions:
* One lane line will likely appear in the left half of the image and the other on the right half.
* The area the binary image that includes a lane line will likely have clusters of '1's.
* Lane lines will form straight or near-straight lines with slight curvatures.  

With the above assumptions in mind, I used a slightly modified version of the sliding window method provided as part of the Udacity Self-Driving Car Nanodegree program. Following is the procedure that I followed:
1. Occurance of 1s of small rectangular subsections(windows) are searched in a binary image.
2. If a subsection includes an above threshold number of ones, the window is concluded to include a lane line.
3. The *mid-point in x-axis of the window* (instead of the point with the most 1s) is taken to be the location of the lane line within the window. This small tweak from the original method prevented the fitted-curve from having odd curvatures.
4. Once (x, y) coordinates of detected lane lines are gathered, best fit curves are generated.

The resulting image is shown below:
![before_warp][./test_video_output/TestOutput.mp4]

### 5. Curvature and Center Calculation
Once the the best fit curves are obtained based on pixel values, we can easily scale them to metric values. I used the values provided by Udacity to do the scaling:   
`ym_per_pix = 30/720 # meters per pixel in y dimension`
`xm_per_pix = 3.7/700 # meters per pixel in x dimension`
Position of vehicle was calculated based on the following assumptions:
    1. The front-facing camera is positioned at the center of the vehicle.
    2. Since 1 is true, each of the two lane lines must be located equidistance away from the left and right edges of an image.
    3. How much the mid-point of detected lane lines  deviate from the mid-point of the lane lines from 2 determines the position of the vehicle respect to center.

The resulting image is shown below:
![before_warp][./report_images/q5_1.png]

### 6. Final Result
![Link to Video][./report_images/q3_1.png]
