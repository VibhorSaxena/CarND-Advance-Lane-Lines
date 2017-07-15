# Advance Lane Detection (Assignment-4)
---

## 1) Description of the pipeline
---

The pipeline can be described in the following phases:- 

1) Correcting Distortion via Chessboard

2) Applying distortion correction to raw images

3) Perspective Transform and creating a thresholded binary image

4) Detecting lane lines in the image

5) Finding the radius of curvature and distance from center

6) Projecting the lane lines to original image to produce final image

---

The individual steps of the pipeline are explained below:-

---

### 1) Correcting Distortion via Chessboard

The sample image of distorted 9x6 chessboard have been used to calculate the distortion matrix. The following image shows the image before and after the correction has been applied to the chessboard.

![Alt text](./images/chessboard_distortion.png?raw=true "Chessboard correction")

---

### 2) Applying distortion correction to raw images

Next, we have applied the same distortion correction to raw images. Here are the results:-

![Alt text](./images/undistort_straight_lines_1.png?raw=true "Straight Lines 1")

![Alt text](./images/undistort_straight_lines_2.png?raw=true "Straight Lines 2")

![Alt text](./images/undistort_test1.png?raw=true "Test 1")

### 3) Perspective Transform and creating a thresholded binary image

I tried the following filters in order to find the binary threshold:-

1) Absolute Sobel threshold

2) HLS thresholds

3) LAB thresholds


The image after a perspective transform and various thresholds looked like following:-

![Alt text](./images/Thresholding.png?raw=true "Test 1")

The final image was constructed using the following combination:-

combined[(((gradx == 1) & (mag_binary == 0))|(s_binary==1)|(la==1) )] = 1

### 4) Detecting Lines in the image

I have implemented a class Line to store the attributes of a line. I have also smoothened the lines by averaging the coefficients used of over 12 previous lines.

I have implemented both methods to detect the lines (in both cases the output is very similar):-

1) Finding Lines from Scratch (by dividing the image into 9 parts and finding the peak in the histogram of white pixels)

![Alt text](./images/findLinesFromScratch.png?raw=true)

2) Look Ahead filter (where we try finding the lines in a region where we found lines previously)

![Alt text](./images/findLinesLookAhead.png?raw=true)

I use the Look Ahead Filter when both the lines in the previous image were found and when the (difference of their radius of curvature)/radius of curvature of left line < 0.5

### 5) Finding Radius of Curvature and Distance from Center

I use the following functions to find radius of curvature and distance from center:-

```
def measure_radius_of_curvature(x_values,num_rows):
    ym_per_pix = 30/720 # meters per pixel in y dimension
    xm_per_pix = 3.7/700 # meters per pixel in x dimension
    y_points = np.linspace(0, num_rows-1, num_rows)
    y_eval = np.max(y_points)
    # Fit new polynomials to x,y in world space
    fit_cr = np.polyfit(y_points*ym_per_pix, x_values*xm_per_pix, 2)
    curverad = ((1 + (2*fit_cr[0]*y_eval*ym_per_pix + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])
    return curverad

def get_intercepts(polynomial):
    bottom = polynomial[0]*720**2 + polynomial[1]*720 + polynomial[2]
    top = polynomial[0]*0**2 + polynomial[1]*0 + polynomial[2]
    return bottom, top

def get_distance_from_center(left_best_fit,right_best_fit):
    left_bottom,left_top=get_intercepts(left_best_fit)
    right_bottom,right_top=get_intercepts(right_best_fit)
    position = (left_bottom+right_bottom)/2
    distance_from_center = abs((640 - position)*3.7/700) 
    return (position,distance_from_center)
```
    
They are used to calculate which line detection function to use and they are also printed on the output image.

### 6) Projecting the lane lines to original image to produce final image

The calculated lines are projected back on the original image using an inverse perspective tranform matrix.

Here is a snapshot of the diagram of how the image flows through the entire process.

![Alt text](./images/flow_diagram.png?raw=true)

## Link to the Final Video

The link to the final video is:- https://youtu.be/CMmFzbFdm-g


## Discussion

### Problems faced in this approach:-

1) The thresholds used in this approach were manually selected basis 5-10 different test images. These could easily fail in different lighting conditions.
2) The approach is not working so well for me in challenge videos as it is detecting another line which is very close to the lane line.

Scenarios where this approach would fail:-

1) First of all, there are many scenarios in which there are no lane markings at all specially on Indian roads. Finding Lane lines would be of no help there.
2) The approach could very easily also fail in different lighting conditions for e.g.- night time or foggy conditions, where lane lines would be hard to see.
3) With varying colors and intensities of lane lines, the thresholds would keep on having to change and hence the thresholds should be adaptive rather than fixed.
# CarND-Advance-Lane-Lines
