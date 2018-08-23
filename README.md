
# **Finding Lane Lines on the Road -pipeline description** 






# Preparation
The following took place :
-Loading and printing Test Images
```python
show_images(images, cmap=None)
```
 -Loding Packages
#  Yellow-White RGB Color masking
Creating  a mask to deploy on images which basicly check which pixels are in the range of white or Yellow 			    	   colors( via RGB boundries ) and keep them while removing the other pixels(who are out of the boundry)
 
```python
select_rgb_color(image)
```
# Canny Edge Detection
## Gray Scailing
Creating Gray Scaling filter which take imge (after the Yellow-White RGB Color masking) and return it with gray values, This is because the Canny edge detection measures the magnitude of pixel intensity changes or gradients.

```
convert_gray_scale(image,color_boundries)
```
## Gaussian Smoothing 
 blurring an image with normal Kernel( in the form of a matrix) in order to reduce noise . We are  Applying the Smoothing on gray scale images.
```
apply_smoothing(image, kernel_size=15)
```
## Edge Detection:
procedure that recives 2 tresholds(upper,lower) and an image (the blured image from previous operation)
looping over all pixals we classify them as followes:

- If a pixel gradient is higher than the upper threshold, the pixel is accepted as an edge
- If a pixel gradient value is below the lower threshold, then it is rejected.
- If the pixel gradient is between the two thresholds, then it will be accepted only if it is connected to a pixel that is above the upper threshold.

We defined them by trials and errors.


```
detect_edges(image, low_threshold=50, high_threshold=150)
```
# Region of Interest Selection
When finding lane lines, we  need  to filter the backround.  
Roughly speaking, we are interested in the area surrounded by the red lines below:

![Region of Interest](http://joe-schueller.github.io/img/region-of-interest.png)

So, we exclude outside the region of interest by apply a mask.

```python
def region_of_interest(img, vertices):
    """
    Applies an image mask.
    
    Only keeps the region of the image defined by the polygon
    formed from `vertices`. The rest of the image is set to black.
    `vertices` should be a numpy array of integer points.
    """
```

## Hough Transform Line Detection
Using `cv2.HoughLinesP` to detect lines in the edge images.

There are several parameters to tweak and tune:

- rho – Distance resolution of the accumulator in pixels.
- theta – Angle resolution of the accumulator in radians.
- threshold – Accumulator threshold parameter. Only those lines are returned that get enough votes (> `threshold`).
- minLineLength – Minimum line length. Line segments shorter than that are rejected.
- maxLineGap – Maximum allowed gap between points on the same line to link them.

We will give a wrapper to `cv2.HoughLinesP`  which output  a list of detected lines(each line represented by two points). 

```
hough_lines(image)
```
which get an image and return list of line (`line=(x1,y1),(x2,y2)`)
```
draw_lines(image, lines, color=[255, 0, 0], thickness=2, make_copy=True)
```
which gets an image and a list of lines and draw the lines on the image.
## Averaging and Extrapolating Lines
### *I   was  helped by  the code in https://github.com/naokishibuya/car-finding-lane-lines of the autor   Naoki Shibuya when I debugged my code in this section*

We want two lane lines: one for the left and the other for the right.  The left lane should have a positive slope, and the right lane should have a negative slope.  Therefore, we'll collect positive slope lines and negative slope lines separately and take averages.

```
average_slope_intercept(lines) #list of lines->left_lane, right_lane
```


### Lines ->Lanes
Using the above `average_lines` function, we can calculate average slope and intercept for the left and right lanes of each image. We still need to convert the slope and intercept into pixel points wewill  define functions to help us with that.



```python
def make_line_points(*args, line):
    """
    Convert a line represented in slope and intercept into pixel points
    """
    return ((x1, y1), (x2, y2))

def lane_lines(image, lines):
    left_lane, right_lane = average_slope_intercept(lines)
    
    .
    .
    .

    left_line  = make_line_points(y1, y2, left_lane)
    right_line = make_line_points(y1, y2, right_lane)
    
    return left_line, right_line
```
### Redefining draw_lines function
Our `draw_lines` except a list of lines as the second parameter.  Each line is a list of 4 values (x1, y1, x2, y2).   List of lines transform to two lines(i.e lanes) and a drawing on the image take place

```
draw_lane_lines(image, lines, color=[255, 0, 0], thickness=20)
```

### 2. Identify potential shortcomings with your current pipeline

Shadows on the lanes may interrupt our  color masking scheme.  as can be observed in in the challenge video 

Only  the straight lane lines are detected and not lines  with curvature.  



### 3. Suggest possible improvements to your pipeline

Transforming color channels from RGB to HSV
In  order to handle curving lines We'll need to use perspective transformation and also poly fitting lane lines rather than fitting to straight lines.

