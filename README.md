# Advanced Lane Finding Project


# Advanced Lane Finding Project

The goals / steps of this project are the following:

- Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
- Apply a distortion correction to raw images.
- Use color transforms, gradients, etc., to create a thresholded binary image.
- Apply a perspective transform to rectify binary image ("birds-eye view").
- Detect lane pixels and fit to find the lane boundary.
- Determine the curvature of the lane and vehicle position with respect to center.
- Warp the detected lane boundaries back onto the original image.
- Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


## Writeup / README

**1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. You can submit your writeup as markdown or pdf.** [**Here**](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) **is a template writeup for this project you can use as a guide and a starting point.**
You're reading it!

## Camera Calibration

**1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.**

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

Image outputs with corner found can be seen under /camera_cal/corners_found*.jpg. Here is an example.

![corners_found1.jpg](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/camera_cal/corners_found1.jpg?raw=true)


I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. 

The code for this step above is under [/camera_cal/can_cal.py.](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/camera_cal/can_cal.py)

With the [calibration_pickle.p](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/camera_cal/calibration_pickle.p) generated by the code linked above, I applied this distortion correction to the test image using the `cv2.undistort()` function in [pipelines.py](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/pipelines.py) line 239.

The results were generated in [/output_images](https://github.com/pineal/Advanced_Lane_Lines_Detection/tree/master/output_images)/[undistort*.jpg](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/output_images/undistort0.jpg)

Here is an example:

![test1.jpg](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/test_images/test1.jpg?raw=true)
![undistort0.jpg](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/output_images/undistort0.jpg?raw=true)


**2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.**

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines 242 in [pipelines.py](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/pipelines.py)) using function color_threshold and abs_sobel_thresh. Here are two example of my output for this step. (note: this is not actually from one of the test images)


![preprocessed0.jpg](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/output_images/preprocessed0.jpg?raw=true)
![preprocessed1.jpg](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/output_images/preprocessed1.jpg?raw=true)


**3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.**

The code for my perspective transform includes a function called `warp_image()`, which appears in lines 99 through 126 in the file  `pipelines.py` . The `warp_image()`function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. I chose the hardcode the source and destination points in the following manner:

        src = np.float32([[[ 610,    450]], 
                          [[ 720,    450]], 
                          [[ w-300,  720]],
                          [[ 380,    720]]])

  

        dst = np.float32([[offset,      0], 
                          [w-offset,    0], 
                          [w-offset,    h], 
                          [offset,      h]])

where h is height of image and w is the width of image, and take an offset of 33% as height.  The binary warped images are were generated in /output_images/warpped*.jpg. Here is an example

![warpped0.jpg](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/output_images/warpped0.jpg?raw=true)


**4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?**

I firstly use fit*_curve() function through line 136 to 162 of pipelines.py to find the curve by window mask and marked them as green like /output_*images/fitted*.jpg:

![fitted0.jpg](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/output_images/fitted0.jpg?raw=true)


Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![](https://github.com/udacity/CarND-Advanced-Lane-Lines/raw/master/examples/color_fit_lines.jpg)


With the output of fix*_curve(), I can also draw the road images like /output_*images/road*.jpg, and uses these points to fit the polynomial curve by numpy.polyfit (line 177 - 180).

**Update**
I added a sanity check according to reviewers sugestion, if the lanes detected are too closed or intersected, I will bypass this result and apply the previous result I got.
**Update End**

**5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.**

From line 177 to line 186 I firstly define conversions in x and y from pixels space to meters where meters per pixel in y dimension is 10/720 and  meters per pixel in x dimension is 4/384.
Then fit new polynomials to x,y in world space:

          left_fit_cr = np.polyfit(ploty*ym_per_pix, leftx*xm_per_pix, 2)
          right_fit_cr = np.polyfit(ploty*ym_per_pix, rightx*xm_per_pix, 2)

  
And get the curverad for both left and right lanes, also use the average to get the center and current position diff form center:


            left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
            right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
            center = (left_fitx[-1] + right_fitx[-1])/2
            center_diff = (warped.shape[1]/2 - center)*xm_per_pix        

**6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.**

Finally I can draw the result back to the road, which was implemented in function draw_image() from line 192 to line 225.
Results can be found in /output_images/tracked*.jpg like:

![tracked0.jpg](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/output_images/tracked0.jpg?raw=true)
![tracked2.jpg](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/output_images/tracked2.jpg?raw=true)

## Pipeline (video)

**1. Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).**

You can find the out videos in the git repository:
[project_video_output.mp4](https://github.com/pineal/Advanced_Lane_Lines_Detection/blob/master/project_video_output.mp4)


## Discussion

**1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?**

In the challenge video tests, I found the result was not performing well when the road covered by shadow or the contrast in the road varies due to some road work. Moreover when sharp curves in sequence happens, it seems the reaction is not good as well. More robust algorithm should be researched to detect the lanes smartly. 

