
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

[image1]: ./output_images/Found_corners.JPG "Corners"
[image2]: ./output_images/Undistorted.JPG "Undistorted"
[image3]: ./output_images/Undistorted_Testimage.JPG "Undistorted Test Image"
[image4]: ./output_images/Thresholded_grad_x.JPG "Gradient X"
[image5]: ./output_images/Thresholded_RGB_B_channel.JPG "Blue channel"
[image6]: ./output_images/Thresholded_LUV_V_channel.JPG "V channel"
[image7]: ./output_images/Thresholded_LAB_B_channel.JPG "B channel"
[image8]: ./output_images/Final_image.JPG "Combined"
[image9]: ./output_images/Bird's-eye-view.JPG "Bird's-eye-view"
[image10]: ./output_images/Lane_lines.JPG "Detected Lane Lines"
[image11]: ./output_images/Detected_lane.JPG "Final Image"

[video1]: ./project_video_processed.mp4 "Video"



### Camera Calibration

The code for this step is contained in the first six code cell of the IPython notebook located in "project.ipynb" 

I started the camera calibration by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z = 0, such that the object points are the same for each calibration image.  

Thus, `objp` is just a replicated array of coordinates, and `objtpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![alt text][image1]

I then used the output `objtpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result on the chessboard:

![alt text][image2]

As a next step I applied the undistortion to a test image:

![alt text][image3]


### Pipeline (single images)

#### 1. Describing how I used color transforms and gradients  to create a thresholded binary image.

I used a combination of color and gradient thresholds to generate a binary image (this step is going through from the 8th to the 26th code cell in the IPhyton notebook.) 

Here's an example of a gradient binary image: 
![alt text][image4]


I used HLS, RGB, LUV, LAB and Gray colorspaces and their channels to create different thresholded binary images that I can use for the image combination. Here are a few examples:

![alt text][image5]
![alt text][image6]
![alt text][image7]

The final output for this step: 

![alt text][image8]

For this image, I ended up using X oriented gradient, RGB, LAB and LUV colorspace images.    


#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `img_warp()`, which appears in the 26th code cell of the IPython notebook.  The `img_warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner:

```python
src = np.array([[img.shape[1]/5.23, img.shape[0]/1.02],  
                [img.shape[1]/2.17, img.shape[0]/1.58],
                [img.shape[1]/1.86, img.shape[0]/1.58],
                [img.shape[1]/1.23, img.shape[0]/1.02]],
                dtype=np.float32)  

dst = np.array([[img.shape[1]/4,img.shape[0]],
                [img.shape[1]/4,0],
                [img.shape[1]/1.33,0],
                [img.shape[1]/1.33,img.shape[0]]],
                dtype=np.float32)
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 245, 706      | 320, 720      | 
| 590, 456      | 320, 0        |
| 688, 456      | 962, 0        |
| 1041, 706     | 962, 720      |


![alt text][image9]

#### 3. Identified lane-line pixels and fit their positions with a polynomial.

In the 30th code cell of the IPython notebook I created a Histogram in order to find pixel value peaks on its left and right side.
These peaks are the starting points for the left and right lines. 

As the next step, I created sliding windows in order to find lane-line pixels in the image. The first sliding windows are placed at the left and right starting points and the next windows will be on top of these. If a window find a minimum pixel values within its window (In my case this value is 50.) it re-centers itself, otherwise it gets the same X position as the last sliding window. 

After this step I got the left and right line pixel positions and created a 2nd order polynomial kinda like this:

![alt text][image10]


#### 4. The radius of curvature of the lane and the position of the vehicle with respect to center.


In the 32nd code cell of the IPython notebook for the curve measurement I used the example code from Udacity's lessons.
For the distance between the Lane and the position of the vehicle, I calculated the center between the two fitted lane lines and measured the distance between this center and the vehicle position. 


#### 5. Example image of of the result.

I implemented this step in the 33rd and 34th code cells. First I used image distortion, then warped the image, detected the lane lines and fitted back the warped image. Here is an example of my result on a test image:


![alt text][image11]


---

### Pipeline (video)


Here's a [link to my video result](./project_video_processed.mp4)

Also in YouTube: [Advanced Lane Finding Project](https://youtu.be/snypdT5v1ig)

---

### Discussion

#### Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The first few step in this project like the camera calibration and the creation of the binary images was straight forward.
The first challenging part in this project for me was the creation of the perspective transform and finding the right values for it.
But, the most challenging part was to find the right combination of the binary images and finding the right threshold values for the lane line detection. It took a long time for me to find these right values for the video.

At this moment the pipeline is only optimized for this video. By combining more binary images for the pipeline detection it could provide a better result. Also the program at this state would most likely fail in a situation where isn't any lane line painted on the road.
