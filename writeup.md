# Advanced Lane Finding Project

The steps of this project are the following:

1. Compute the camera calibration matrix
2. Apply distortion coefficients given a set of test images.
3. Apply a perspective transform to rectify binary image ("birds-eye view").
4. Use color transforms, gradients, etc., to create a thresholded binary image and detect lane pixels.
5. Fit to find the lane boundary, determine the curvature of the lane and vehicle position with respect to center, and warp the detected lane boundaries back onto the original image.
6. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/draw_chessboard_corners.png "Chessboard Corners"
[image2]: ./output_images/undistort_example.png "Undistorted"
[image3]: ./output_images/warped_example.png "Warped"
[image4]: ./output_images/binary_thresholds_example.png "Binary Trhesholds"
[image5]: ./output_images/warp_back.png "Warp Back"
[image6]: ./output_images/project_result.gif "Result"
[image7]: ./output_images/challenge_result.gif "Challenge Result"
[image8]: ./output_images/harder_challenge_result.gif "Harder Challenge Result"

### 1. Camera Calibration

The code for this step is contained in the first code cell of the IPython notebook located in `process.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

![][image1]

### 2. Dirstortion Correction

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the highway driving test image using the `cv2.undistort()` function.

![][image2]

As you can see above, there are obvious differences between the original image and undistorted image on the top-left.

### 3. Birds-Eye View

The code for my perspective transform includes a function called `warped()`, which appears in 5th code cell of the IPython notebook. The `warped()` function takes as inputs an image (`image`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
offset = 30
corx1  = 0.38
corx2  = 0.63
cory   = 0.67

src = np.float32([[img_size[0]*corx1, img_size[1]*cory],
                  [img_size[0]*corx2, img_size[1]*cory],
                  [img_size[0]-offset, img_size[1]-offset],
                  [offset, img_size[1]-offset]])

dst = np.float32([[offset, offset],
                  [img_size[0]-offset, offset],
                  [img_size[0]-offset, img_size[1]-offset],
                  [offset, img_size[1]-offset]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 486, 482      | 30, 30        |
| 806, 482      | 1250, 30      |
| 1250, 690     | 1250, 690     |
| 30, 690       | 30, 690       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![][image3]

### 4. Apply Binary Thresholds and Detect Lane Pixels

I used a combination of color and gradient thresholds to generate a binary image. (thresholding steps in 23rd code cell of the IPython notebook). I try to test all of methods that I learned in the class, such as magnitude, direction, RGB and HLS channel. I chose that the following methods and thresholds due to work well.

* The magnitude, with  kernel size of `9`, a min threshold of `15`, and a max threshold of `70`.
* The H channel from HLS color space, with a min threshold of `19` and a max threshold of `60`. I expected that the intersection of magnitude and H channel would classify yellow lane lines with shadows on the road.
* The R channel from RGB color space, with a min threshold of `230` and a max threshold of `255`. It makes clear most of yellow lane lines.
* The B channel from RGB color space, with a min threshold of `200` and a max threshold of `250`.
* The L channel from HLS color space, with a min threshold of `220` and a max threshold of `255`. Both the B cannel and the L channel can show white lane lines clearly.

I created a combined binary threshold using by 5 methods like this:

```python
mag_and_h_channel[(magnitude == 1) & (h_channel == 1)] = 1
combined[(mag_and_h_channel == 1) | (r_channel == 1) | (b_channel == 1) | (l_channel == 1)] = 1
```

![][image4]

### 5. Fitting a Polynomial to Lane Lines, Calcuating Radius of Curvature and Vehicle Position, and Warp Back

For fitting a polynomial to left and right lane line. First, I checked peaks in a histogram of the image to determine location of both lane lines, and processed all non zero pixels around histogram peaks (these steps in 24th code cell of the IPython notebook). At last, fit a polynomial to left and right lanes using by `np.polyfit()` (this step in 25th code cell of the IPython notebook).

After previous job, it was possible to calculate vehicle position with respect to center like below:

```python
# rightx_int and leftx_int are the interception of polinomials
position = (rightx_int+leftx_int)/2
distance_from_center = abs((640 - position)*3.7/800)
```

I then calcuated radius of curature using by following code:

```python
ym_per_pix = 30./720
xm_per_pix = 3.7/800
# Must do each lane lines
fit_cr = np.polyfit(yvals*ym_per_pix, xvals*xm_per_pix, 2)
curverad = ((1 + (2*fit_cr[0]*np.max(yvals) + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])
```

Fitting a polynomial to the lane lines and warp the detected lane boundaries back onto the original image look like below:

![][image5]

### 6. Output Visual Display

At last, in order to show the result smoothly, I setted up to remember the average of coefficients of previous 10 steps' polynomial. Here's a [link to my video result](./project_result.mp4) and a GIF file below:

![][image6]


Discussion
---

| Challenge   | Harder Challenge  |
|:-----------:|:-----------------:|
| ![][image7] | ![][image8]       |

As you can see above images, my results have problems to make output using by challenge works. In the Challenge part, I need to research about another binary thresholds with color channel, such as `LUV` or `Lab`. But, Harder Challenge should think more about drawing lane lines when the vehicle turns hard right (from 37s to 42s on the video). I ought to rethink about left and right lane lines that I divided by half on image, but I have to reconstruct the center line of image when the vehicle turns.
