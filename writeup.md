# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

At the most basic level, my pipeline follows the approach of the Hough exercise in the class.

1. Grayscale the image.
2. Run Gaussian blur on the image.
3. Find edges using Canny edge detector.
4. Mask edges with isosceles trapezoidal region of interest
5. Find line segments with OpenCV Hough line transform
6. Draw lines from the Hough line segments

For drawing single left/right lane lines, and for the challenge video, I created a
second Hough function for drawing single left and right lane lines called 'hough\_single\_lines'.
'hough\_single\_lines' calls a helper function to compute the single right and left lanes,
and uses the provided 'draw\_lines' function to draw those lines.  I left all the helper
functions as they are.

The logic for drawing single lane lines on each side is to take the Hough line segments and divide them into collections of left and right lane line points. This was done as follows

1. For each Hough line segment, compute its slope
2. Otherwise, use the slope to determine if it is a left lane line (slope < 0) or right lane line.
3. For each side, add the endpoints of the line segment into a collection each of x and y coordinates.  At the end, there are four collections - left x, left y, right x, and right y.
4. From the left x/y coordinates compute the left lane line, and from the right x/y coordinates,
compute the right lane line.

The single lane line for each side is computed by fitting a 1-degree polynomial to the
point collections using numpy polyfit.  From there, the lane line is computed taking the top
and bottom edges of the region of interest as the y coordinates, and using the polynomial
fit to compute the corresponding x coordinates.  This is done in the 'average\_line' function.

Finally, for the challenge video, the above logic is modified in 3 ways.

1. First, a moving average from past video frames is used to smooth out the line from frame to frame.
2. Second, for the Hough line segments, if the line slope is too small or too large, ignore the line
2. Third, as new frames are processed, if the new average line has a slope that changes
too much from the current average, the average line is discarded and the moving average is used
instead.

For the moving average, a fixed window queue of the computed average line of each frame is used.
A queue is used instead of a simple numerical moving average to help smooth the average as the first set of frames are processed. In practice, a moving average or at least remembering the last
good lane line is important as some frames in the challenge video require throwing out all the lines due to extreme slopes or high slope changes. So for some frames, we just keep the last known lane line.

The implementation handles edges cases where no lane lines are computed.


### 2. Identify potential shortcomings with your current pipeline

A shortcoming of the current approach is that it will not work very well with
roads with high curvature, because we draw single straight lane lines
for each side.

A potential shortcoming is that the x and y coordinates of the lines for either side are
combined into a single collection without regard to whether it is the 'upper' or 'lower'
end of the line.  Finding centroids of top and bottom coordinates may yield different results.

Another shortcoming could be that line segments are not checked for whether they cut across
the middle of the image.

It is unclear how well the implementation works on different lighting and
weather conditions such as night time, sundown, clouds, and rain.  

Finally, the algorithm may not handle the car changing lanes, exiting freeway, and with other cars
navigating in the field of view.

### 3. Suggest possible improvements to your pipeline

A possible improvement could be to dynamically adjust the region of interest
based on the curvature of the road.

A good improvement would be to handle changing of lanes, including using lane finding
to determine when to stop changing lanes.

