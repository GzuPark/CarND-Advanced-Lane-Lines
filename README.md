## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


Steps
---

The steps of this project are the following:

1. Compute the camera calibration matrix
2. Apply distortion coefficients given a set of test images.
3. Apply a perspective transform to rectify binary image ("birds-eye view").
4. Use color transforms, gradients, etc., to create a thresholded binary image and detect lane pixels.
5. Fit to find the lane boundary, determine the curvature of the lane and vehicle position with respect to center, and warp the detected lane boundaries back onto the original image.
6. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The images for camera calibration are stored in the folder called `camera_cal`. The images in `test_images` are for testing your pipeline on single frames.  If you want to extract more test images from the videos, you can simply use an image writing method like `cv2.imwrite()`, i.e., you can read the video in frame by frame as usual, and for frames you want to save for later you can write to an image file.  

Files
---
* `writeup.md`: Describe the project what I apply tools and techniques
* `process.ipynb`: Code following by steps of pipeline
* `*_video.mp4`: Videos to make result for the project
* `camera_cal`: Images for camera calibration
* `output_images`: Images for writeup
* `test_images`: Highway images for testing pipeline
