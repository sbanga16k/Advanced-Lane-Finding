
# Udacity

## Advanced Lane Finding Project

The goals / steps of this project are the following:
1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
2. Apply a distortion correction to raw images.
3. Apply a perspective transform to rectify undistorted image ("birds-eye view").
4. Use color transforms, gradients, etc., to create a thresholded binary image.
5. Detect lane pixels and fit to find the lane boundary.
6. Determine the curvature of the lane and vehicle position with respect to center.
7. Warp the detected lane boundaries back onto the original image.
8. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./disp_images/img_dist_correction.jpg "Distortion-corrected calibration images"
[image2]: ./disp_images/img_test_car.jpg "Car test image"
[image3]: ./disp_images/img_dist_correction_car.jpg "Distortion-corrected car test image"
[image4]: ./disp_images/img_binary.jpg "Binary thresholded test image"
[image5]: ./disp_images/img_warped1.jpg "Original image - source points drawn"
[image6]: ./disp_images/img_warped2.jpg "Warped image"
[image7]: ./disp_images/img_binary_hist.jpg "Test image binary activation histogram"
[image8]: ./disp_images/img_lanes_visuals.jpg "Lane visualization sliding window"
[image9]: ./disp_images/img_lane_prevfit.jpg "Lane search previous fit"
[image10]: ./disp_images/img_final.jpg "Lane projected onto image"

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation


### Writeup / README

1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. You can submit your writeup as markdown or pdf. Here is a template writeup for this project you can use as a guide and a starting point.
You're reading it!

### Camera Calibration

1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
The code for this step is contained in the second code cell of the Jupyter notebook "Adv_Lane_finding.ipynb".

I start by preparing "object points", which will be the (x, y, z) world coordinates of the chessboard corners. I make the assumption that the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each of the calibration images. Thus, 'obj_p' is just a replicated array of coordinates, and 'obj_pts' will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. 'img_pts' will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard corner detection.

I then used the output obj_pts and img_pts to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction using the cv2.undistort() function to a few calibration images and obtained the following result:

![alt text][image1]

### Pipeline (single images)

For all of the following steps, I have chosen to visualize the various steps of the pipeline using the following test image:

![alt text][image2]

1. Provide an example of a distortion-corrected image.
The result obtained after distortion correction of the test image is depicted below: 

![alt text][image3]

The effects of distortion correction are not very evident in the image except for on the hood of the car appearing on the bottom edge of the image.

2. Describe how you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

I experimented with various methods of thresholding for obtaining the binary image with the lane lines captured clearly and robust enough to be unaffected by random shadows cast by trees. These are listed below and their outputs can be seen in the jupyter notebook:
* Absolute x & y-gradient thresholding (Useful for detecting edges of lane lines)   --- Code cell 6
* Gradient magnitude thresholding (To suppress spurious edges)                      --- Code cell 7
* Gradient direction thresholding (Since lane lines are nearly vertical, atleast near the bottom of the image)  --- Code cell 8
* Color thresholds for different color spaces: RGB, HSV,, HLS, LAB, LUV             --- Code cell 10

After visulaizing the outputs of each of the channels for the aforementioned color channels, it became evident that some channels did a much better job at identifying different colored lane lines than others:
1. 'B' channel of 'LAB' colorspace detected yellow lane lines robustly without any noise    --- Code cell 11
2. 'L' channel of 'LUV' colorspace detected white lane lines robustly without much noise    --- Code cell 12

Therefore, a combination of the above two channels and x-gradient absolute thresholding were used to capture lane lines in images clearly. The output is depicted below:

![alt text][image4]

3. Describe how you performed a perspective transform and provide an example of a transformed image.
The code for my perspective transform includes a function called 'warp_img', which appears in the 14th code cell of the IPython notebook. The 'warp_img' function takes as inputs an image (img), as well as source (src) and destination (dst) points. 

I chose to hardcode the source and destination points. This resulted in the following source and destination points:

Source	Destination
500, 475	100, 0
800, 475	1180, 720
1250, 50	1180, 0
50,  720	100, 720

I verified that my perspective transform was working as expected by drawing the src and dst points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image as shown below:

![alt text][image5]
![alt text][image6]

4. Describe how you identified lane-line pixels and fit their positions with a polynomial?
I used the histogram of binary activations for the bottom quarter of the binary warped image to identify the lane lines corresponding to the peaks in the histogram to the left and right of the vertical half of the image.  --- Code cell 18

The histogram for the test image is shown below:
![alt text][image7]

I then proceeded to identify pixels corresponding to the lane lines as those which had a value of 1 in the binary image and lay within a certain margin (say, 80 pixels) of the mean of the existing pixels corresponding to each lane line.  --- Code cell 19

This was accomplished by implementing a sliding window approach which searched the image to identify the lane pixels from scratch and then fitting a 2nd order polynomial to fit their positions.    --- Code cells 19-20

![alt text][image8]

I also implemented a sliding window approach method which used information about the fit obtained from the previous frame to limit the search area for identifying lane pixels and then fit a 2nd polynomial to these lane positions.  --- Code cells 21-22

The search area is illustrated with the help of the following image:

![alt text][image9]

5. Describe how you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
The radius of curvature for the lane lines was calculated using their corresponding polynomial fits.    --- Code Cell 24
The vehicle position was computed using the middle of the image and the pixels corresponding to the lane lines at the bottom of the image.

6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
The code for plotting the polygon corresponding to the area between the lanes and printing the text with radius of curvature information & vehicle position relative to the lane center is written in code cells 26-29 of the Ipython notebook. 

An example image is depicted below:

![alt text][image10]

### Pipeline (video)
1. Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).
The code for creating a class for storing attributes for each of the lanes is written in code cell 30 of Ipython notebook. The code for processing the frames in the video in code cell with label 42.
The video file:
```sh
'project_video_output.mp4'
```
is included in the main folder and is also in the Ipython notebook code cell with label 44. 

### Discussion
1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?
I used a combination of absolute gradient threshold and color thresholds from different colorspaces to identify lane lines in images. I also used sliding fit approach for identifying pixels belonging to each lane line and a class to keep track of the mean positions of the lane lines and their fits for a few previous frames.
The sliding fit approach is likely to fail when there is ambiguity about the boundary of lane lines with a dark patch within the lane line and when the lane is in complete shade for a while. 
This could potentially be overcome by enforcing a condition of enforcing a constant width of the lane and then identifying the true lane lines. There might also exist a possibility of extracting features corresponding to lane lines from images that might be more robust to changes in lighting conditions.
