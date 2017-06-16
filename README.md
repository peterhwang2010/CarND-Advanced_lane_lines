[//]: # (Image References)

[image1]: ./examples/original.png "Original Image"
[image2]: ./examples/undistorted.png "Undistorted"
[image3]: ./examples/perspective_transform_img.jpg "Perspective Transformed Image"
[image4]: ./examples/l-channel.png "L Channel"
[image5]: ./examples/s-channel.png "S Channel"
[image6]: ./examples/combine-channel.png "Combine Channel"
[image7]: ./examples/polynomial-fitted.png "Polynomial Fitted"
[image8]: ./examples/polynomial-drawn.png "Polynomial drawn"
[image9]: ./examples/highlighted-lane.png "Highlighted Lane"
[video1]: ./project_video.mp4 "Video"

# Advance Lane Lines 

## Camera Calibration

To calibrate the camera, I used the images of chessboard provided to me. I used the `cv2.findChessboardCorners` to find the corners of the board. Once I found the corners of the chessboard, I drew the lines using `cv2.drawChessboardCorners`.

After finding the image points and objects points, I undistorted the camera lens.

## Undistorting Image

After I finished calibrating the camera, I undistorted the image by using the object and image points from previous function. I called the `cv2` function called `cv2.undistort` and passed in the image, image points, and obj points as the parameters, which return an image that has been undistorted.

### Original Image 
![alt text][image1]

### Undistorted Image 
![alt text][image2]

## Perspective Transform 

With the undistorted image, I performed the perspective transform on the image. The point of the perspective transform is to view image as the bird's eye view. Which will be useful later to calculate the lane curvature. 

The way I performed the perspective transform was by using `cv2` library. We called on the `cv2.getPerspectiveTransform` and passed in the destination and source points. The source points the I used for this project ...

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 490, 482      | 0, 0          | 
| 810, 482      | 1280, 0       |
| 1250, 720     | 1250, 720     |
| 40, 720       | 40, 720       |

### Perspective Transform Code
```
  def transform_perspective (img, src_coords, dst_coords):
      M = cv2.getPerspectiveTransform(src_coords, dst_coords)
      
      return cv2.warpPerspective(img, M, (img.shape[1],img.shape[0]), flags=cv2.INTER_NEAREST)
```

### Original Image 
![alt text][image1]

### Perspective Transformed Image 
![alt text][image3]

## Color Thresholding

In order to detect the lane lines better I used the saturation and lightness channel.  After applying the threshold on both channels, I combined the two to determine the lane lines in the image.

### S Channel
![alt text][image4]

### Code
```
  def s_channel(img, threshold_min, threshold_max):
      s_channel = cv2.cvtColor(img, cv2.COLOR_BGR2HLS)[:,:,2]
      s_binary = np.zeros_like(s_channel)
      s_binary[(s_channel >= threshold_min) & (s_channel <= threshold_max)] = 1
      return s_binary
```
### L Channel
![alt text][image5]

### Code
```
  def l_channel(img, threshold_min, threshold_max):
      l_channel = cv2.cvtColor(img, cv2.COLOR_BGR2LUV)[:,:,0]
      l_binary = np.zeros_like(l_channel)
      l_binary[(l_channel >= threshold_min) & (l_channel <= threshold_max)] = 1
      return l_binary
```
### Combine Channel
![alt text][image6]

### Code
```
  def combine_binary(s_binary, l_binary):
      binary = np.zeros_like(s_binary)
      binary[(l_binary == 1) | (s_binary == 1)] = 1
      return binary
```

## Identifying the Lane Lines 

The way I identifies the lane lines is by first dividing `n` steps by equal height. And for each step I generated the histogram, and smoothened the histogram using `scipy` library and calling the function called `signal.medfilt`. Once finished smotthing the histogram, I looked for the peak on the left half and also on the right half. Finally we added the pixels to the collections of the lane line pixels . Than we used the collections to fit the polynomials to each lane by using `np.polyfit`.

### Polynomial Fitted 
![alt text][image7]

### Polynomial Drawn
![alt text][image8]
### Code 
```
  def draw_poly(img, poly, poly_coeffs, steps, color=[255, 0, 0], thickness=10, dashed=False):
    img_height = img.shape[0]
    pixels_per_step = img_height // steps

    for i in range(steps):
        start = i * pixels_per_step
        end = start + pixels_per_step

        start_point = (int(poly(start, poly_coeffs=poly_coeffs)), start)
        end_point = (int(poly(end, poly_coeffs=poly_coeffs)), end)

        if dashed == False or i % 2 == 1:
            img = cv2.line(img, end_point, start_point, color, thickness)

    return img
```

### Lane Lines Highlighted
![alt text][image9]
### Code
```
  def highlight_lane_line_area(mask_template, left_poly, right_poly, start_y=0, end_y =720):
      area_mask = mask_template
      for y in range(start_y, end_y):
          left = evaluate_poly(y, left_poly)
          right = evaluate_poly(y, right_poly)
          area_mask[y][int(left):int(right)] = 1
      
      return area_mask
```