[//]: # (Image References)

[image1]: ./examples/original.png "Original Image"
[image2]: ./examples/undistorted.png "Undistorted"
[image3]: ./examples/perspective_transform_img.jpg "Perspective Transformed Image"
[image4]: ./examples/l-channel.png "L Channel"
[image5]: ./examples/s-channel.png "S Channel"
[image6]: ./examples/combine-channel.png "Combine Channel"
[video1]: ./project_video.mp4 "Video"

# Advance Lane Lines 
---

## Camera Calibration
---

To calibrate the camera, I used the images of chessboard provided to me. I used the `cv2.findChessboardCorners` to find the corners of the board. Once I found the corners of the chessboard, I drew the lines using `cv2.drawChessboardCorners`.

After finding the image points and objects points, I undistorted the camera lens.

## Undistorting Image
---

After I finished calibrating the camera, I undistorted the image by using the object and image points from previous function. I called the `cv2` function called `cv2.undistort` and passed in the image, image points, and obj points as the parameters, which return an image that has been undistorted.

### Original Image 
![alt text][image1]

### Undistorted Image 
![alt text][image2]

## Perspective Transform 
---

With the undistorted image, I performed the perspective transform on the image. The point of the perspective transform is to view image as the bird's eye view. Which will be useful later to calculate the lane curvature. 

The way I performed the perspective transform was by using `cv2` library. We called on the `cv2.getPerspectiveTransform` and passed in the destination and source points. The source points the I used for this project ...

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 490, 482      | 0, 0          | 
| 810, 482      | 1280, 0       |
| 1250, 720     | 1250, 720     |
| 40, 720       | 40, 720       |

### Original Image 
![alt text][image1]

### Perspective Transformed Image 
![alt text][image3]

## Color Thresholding
---

Inorder to detect the lane lines better I am using saturation and lightness channel.  After applying the threshold on both channels, I am combining the two to determine the lane lines in the image.

### S Channel
![alt text][image4]

### L Channel
![alt text][image5]

### Combine Channel
![alt text][image6]
