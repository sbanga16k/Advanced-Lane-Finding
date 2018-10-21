
## Advanced Lane Finding Project

The goals of this project are the following:
1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
2. Apply a distortion correction to raw images.
3. Use color transforms, gradients to create a thresholded binary image.
4. Apply a perspective transform to rectify undistorted image ("birds-eye view").
5. Detect lane pixels and fit to find the lane boundary.
6. Determine the curvature of the lane and vehicle position with respect to center.
7. Warp the detected lane boundaries back onto the original image.
8. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


### Step 1: Camera Calibration

Firstly, the camera matrix and distortion coefficients is computed using calibration images. 

I start by preparing "object points", which are the (x, y, z) world coordinates of the chessboard corners. I make the assumption that the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each of the calibration images. Thus, 'obj_p' is just a replicated array of coordinates, and 'obj_pts' is appended with a copy of it every time after successful detection of chessboard corners in a test image. 'img_pts' will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard corner detection.

I then use the output 'obj_pts' and 'img_pts' to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction using the cv2.undistort() function to a few calibration images and obtained the following result:

![Alt Text](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/output_images/img_dist_correction.JPG?raw=true "Distortion-corrected calibration images")


## Pipeline (single image)

### Step 2: Distortion correction of images

For all of the following steps, I chose to visualize the various steps of the pipeline using the following test image:

![Alt Text](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/output_images/img_test_car.JPG?raw=true "Car test image")

The result obtained after distortion correction of the test image is depicted below: 

![Alt Text](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/output_images/img_dist_correction_car.JPG?raw=true "Distortion-corrected car test image")

The effects of distortion correction are not very evident in the image except for on the hood of the car appearing on the bottom edge of the image.

### Step 3: Generating thresholded binary image

I experimented with various methods of thresholding for obtaining the binary image with the lane lines captured clearly and robust enough to be unaffected by random shadows cast by trees. These are listed below and their outputs can be seen in the jupyter notebook:
* Absolute x & y-gradient thresholding (Useful for detecting edges of lane lines)   --- Code cell 6
* Gradient magnitude thresholding (To suppress spurious edges)                      --- Code cell 7
* Gradient direction thresholding (Since lane lines are nearly vertical, atleast 
  near the bottom of the image)                                                     --- Code cell 8
* Color thresholds for different color spaces: RGB, HSV,, HLS, LAB, LUV             --- Code cell 10

After visulaizing the outputs of each of the channels for the aforementioned color channels, it became evident that some channels did a much better job at identifying different colored lane lines than others:
1. 'B' channel of 'LAB' colorspace detected yellow lane lines robustly without any noise    --- Code cell 11
2. 'L' channel of 'LUV' colorspace detected white lane lines robustly without much noise    --- Code cell 12

Therefore, a combination of the above two channels and x-gradient absolute thresholding were used to capture lane lines in images clearly. The output is depicted below:

![Alt Text](https://github.com/sbanga16k/Advanced-lane-finding/blob/master/output_images/img_binary.JPG?raw=true "Binary thresholded test image")

### Step 4: Image rectification ("bird's-eye view" transform)

The code for my perspective transform is encapsulated inside a function called `'warp_img'`. The function takes as input an image (img), as well as source (src) and destination (dst) points which represent mapping from original to bird's-eye view perspective using 4 pairs of points.

I chose to hardcode the source and destination points since the position of the car with respect to the lane remained constant throughout the video stream. This resulted in the following source and destination points:

Source      Destination
500, 475    100, 0
800, 475    1180, 720
1250, 50    1180, 0
50,  720    100, 720

I validated the correctness of my perspective transform implementation by drawing the src and dst points onto a test image and its warped counterpart to verify if the lane lines appear parallel in the warped image. This is depicted below:

![Alt Text](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/output_images/img_warped1.JPG?raw=true "Original image - source points drawn")

![Alt Text](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/output_images/img_warped2.JPG?raw=true "Warped image")

### Step 5: Detection of lane pixels and fitting polynomial to lane lines

I used the histogram of binary activations for the bottom quarter of the binary warped image to identify the lane lines corresponding to the peaks in the histogram to the left and right of the vertical half of the image.

The histogram for the test image is shown below:
![Alt Text](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/output_images/img_binary_hist.JPG?raw=true "Test image binary activation histogram")

Pixels corresponding to the lane lines were identified as those with a value of 1 in the binary image and lying within a certain margin (say, 80 pixels) of the mean of the existing pixel coordinates corresponding to each lane line.

This was accomplished by implementing a sliding window approach which searched the image to identify the lane pixels from scratch and then fitting a 2nd order polynomial to fit their positions.                 --- Code cells 19-20

![Alt Text](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/output_images/img_lanes_visuals.JPG?raw=true "Lane visualization sliding window")

I also implemented a more 'intelligent' sliding window method which used information about the fit obtained from the previous frame to limit the search area for identifying lane pixels and then fit a 2nd polynomial to these lane positions.  
                                                                                        --- Code cells 21-22

The search area is illustrated with the help of the following image:

![Alt Text](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/output_images/img_lane_prevfit.JPG?raw=true "Lane search previous fit")


### Step 6: Calculation of lane curvature and position of the vehicle relative to lane center

The radius of curvature for the lane lines was calculated using their corresponding polynomial fits.
The vehicle position was computed using the middle of the image and the pixels corresponding to the lane lines at the bottom of the image.

### Step 7: Projection of lane lines back onto original image perspective

The code for plotting the polygon corresponding to the area between the lanes and printing the text with radius of curvature information & vehicle position relative to the lane center is written in code cells 26-29 of the Ipython notebook. 

An example image is depicted below:

![Alt Text](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/output_images/img_final.JPG?raw=true "Lane projected onto image")


## Pipeline (video)

A class was implemented for lane detection for video stream. It served to store attributes related to coefficients representing polynomial fit for each of the lanes and the mean coordinates for the lanes. The code for processing the frames in the video in code cell 42. It stacks the various elements of the pipeline discussed above and employs the 'intelligent' sliding window implementation to exploit information about lane line detections in previous frames of the video.

The video file is generated using `VideoFileClip` function of `moveipy.editor` module. It is embedded here as:

[![Watch the video](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/Video_output_thumbnail.png?raw=true)](https://github.com/sbanga16k/Advanced-Lane-Finding/blob/master/project_video_output.mp4)

### Discussion

I used a combination of absolute gradient threshold and color thresholds from different colorspaces to identify lane lines in images. I also used sliding fit approach for identifying pixels belonging to each lane line and a class to keep track of the mean positions of the lane lines and their fits for a few previous frames.
The sliding fit approach is likely to fail when there is ambiguity about the boundary of lane lines with a dark patch within the lane line and when the lane is in complete shade for a while. 
This could potentially be overcome by enforcing a condition of enforcing a constant width of the lane and then identifying the true lane lines. There might also exist a possibility of extracting features corresponding to lane lines from images that might be more robust to changes in lighting conditions.
