## Writeup

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

[image1]: ./output_images/calib_test.jpg "calib_test"
[image2]: ./output_images/img_undist.jpg "img_undist"
[image3]: ./output_images/img_thresholded_binary.jpg "img_thresholded_binary"
[image4]: ./output_images/img_perspective_trans.jpg "img_perspective_trans"
[image5]: ./output_images/lane_boundary.jpg "lane_boundary"
[image6]: ./output_images/masked_img.jpg "masked_img"
[image7]: ./output_images/final_img2.jpg "final_img2"
[image8]: ./output_images/final_img1.jpg "final_img1"
[image9]: ./output_images/final_img3.jpg "final_img3"
[image10]: ./output_images/final_img4.jpg "final_img4"
[image11]: ./output_images/final_img5.jpg "final_img5"
[image12]: ./output_images/final_im6.jpg "final_img6"
[video11]: ./project_video.mp4 "Video"


### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. 

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second and third code cell of the jupyter notebook called `Advanced_Lane_Line_Detection_image.ipynb`(also the same location in `Advanced_Lane_Line_Detection_video.ipynb`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how to create a thresholded binary image.

I used a combination of color and gradient thresholds to generate a binary image. 

First, I choosed the gradient of x axis with a maximum threshold of 100 and minimum threshold of 30; Second, I choosed the absolute gradient magnitude with a maximum threshhold of 100 and a minimum threshold of 30. Besides, I used the S channel of the HLS image with a threshold range from 0.7 to 1.3,because this channel could be pretty robust against shadows and lights. At last, I created a combined mask to keep the pixels which satisfied these conditions, and set these pixels as 1 while the others as 0. The combined mask image was the thresholded binary image i wanted.

Here's an example of a binary image result.

![alt text][image3]

#### 3. Describe how I performed a perspective transform.

The code for my perspective transform includes a function called `birds_eye_transfer()`, which appears in the 5th cell of the file `Advanced_Lane_Line_Detection_image.ipynb` .  The `birds_eye_transfer()` function takes as inputs an image (`img`), as well as a (`flag`) .  

If the flag equals 1, the function will transform an binary image into its bird-eye view, if the flag equals 2, the function will transform an bird-eye view into the normal view.

I chose the hardcode the source and destination points in the following manner:

```python
offset = 90
src = np.float32([[559,477],[280,680],[1035,680],[728,477]])
dst = np.float32([[280+offset,330],[280+offset,680],
                  [1035-offset,680],[1035-offset,330]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 559, 477      | 380, 330        | 
| 280, 680      | 380, 680      |
| 1035, 680     | 935, 680      |
| 728, 477      | 935, 330        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how  I identified lane-line pixels and fit their positions with a polynomial.

Then I found the start points of each line based on a histogram of vertical pixels of the binary image.

After that, starting from the start points, I used the sliding windows method to find all the points in the sliding windows.

At last, I fit my lane lines with a 2nd order polynomial with these points, kinda like this:

![alt text][image5]

#### 5. Describe how I calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the 5th cell of `Advanced_Lane_Line_Detection_image.ipynb` The function was `get_curvature()`,located in the line 176. In the video pipeline, I did this inside the class `Line` with the function `set_radius_of_curvature()`

#### 6.An example image of the result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 5th cell of `Advanced_Lane_Line_Detection_image.ipynb` in the function `show_boundaries()` and `visual_display()`.  Here is an example of my result on a test image:

![alt text][image7]


---

### Pipeline (video)

#### 1. Provide a link to the final video output.  

Here's a [link to my video result](./processed_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, I used a buffer to store several frames of detection results, so if the algorithm fails the current frame, i can use an averaged results in the buffer as the result of the current frame. Besides, if a polynomial fit was found to be robust in the previous frame, then rather than search the entire next frame for the lines, just a window around the previous detection could be searched. This will improve speed and provide a more robust method for rejecting outliers. These methods are really helpful when there are shadows or unclear lane markings, or bumps of the vehicle, where the traditional method we use to extract the lane marking features usually fail.

There are two locations in the video where we got bumps of the vehicle, and the pipeline performed less robust than the other locations. If we lower the fps of the video, the pipeline might fail. This is because the speed of the vehicle is pretty high, the image scenes change a lot, the averaging results of the buffer could not keep up the changes in time.

To improve the pipeline, a more reliable lane line feature extraction method,such as DNN algorithms, might help a lot, besides, an adaptive buffer might also help, so we can use adaptive strategies to buffer information based on current speed,road conditions,specific detection results,etc.
