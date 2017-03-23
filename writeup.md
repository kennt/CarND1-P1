# CarND1-P1
# **Finding Lane Lines on the Road** 

[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[imageMask]: ./examples/mask.jpg "Trapezoidal mask"
[imageShortWhiteLine]: ./examples/short_white_line.jpg "Short white line"

---

### Reflection

### 1. Image Pipeline

My initial pipeline consists of 3 main steps:

1. **edge detection** (`detect_edges`)

	Performs edge detection on the image.  Runs through a 3-step process.

	a. **gray-scale conversion**

	b. **gaussian blur**

	c. **Canny edge detection**

2. **line detection** (`hough_lines`)

	Use the Hough transform for line detection.  This returns a set of lines to be used for the final image.

3. **the final image generation** (`draw_lines`)

	Eliminates extraneous lines from the image and generates an image to be layered on top of the initial image.

This follows the examples from the quizzes.  

Step (3), line detection, inside the `draw_lines` function, contains the changes made to exclude certain lines from the image. The first part is to exclude edges that are not in the bottom part of the image, this is a trapezoidal area.

![alt text][imageMask]

The lines are then searched for using the Hough Transform using edges within this region.  The lines that are excluded include:

* horizontal lines
* close-to-vertical lines
* lines that do not go through the bottom of the image
* left lines that do not go through left-half bottom of the image
* left lines that do not go through the top right
* right lines that do not go through the top left
* right lines that do not go through right-half bottom of the image

The purpose of this pruning is to exclude extraneous lines (that may come from other edges on the road).

In addition, there was some experimentation with the Hough Transform parameters, especially the `min_line_length` and `max_line_gap`.  The `min_line_length` had to be adjusted to work with the dashed lines.  This is especially true when the closest line goes out of view (see image below).

![alt text][imageShortWhiteLine]

This initial version worked well for the white-stripe and yellow-strip videos, so nothing else needed to be done for those videos.

But there was some difficulty with the challenge video. It was clear that the edge detection was having difficulty with detecting the edges on the concrete portion of the highway (not enough contrast).  I ran some experiments and found that using the 'V' channel from the HSV rather than RGB, gave better results.  This was implemented in the `detect_yellow_edges` function.

```python
    hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
    s = hsv[:,:,2] 
```

I also experimented with merging the results from the RGB and HSV line detection results (which is why I separated the drawing of lines from the Hough-transform line detection code).  Merging the two results seems to result in smoother lane lines (it didn't jump around as much), but required more processing (took about 20% longer on the challenge video).

So in the end, instead of converting to gray scale, I used the V channel after converting to HSV.


### 2. Potential shortcomings

Given the difficulties of the challenge video, running the code on a concrete highway (rather than an asphalt highway) may be a problem.

Lines from non-road sources could cause a problem (say from other vehicles).

Detecting lines from dotted lane markers.  Not all lanes are marked with lines, they may be using dotted lane markers (such as '...  ...  ...  ...').

Detecting lines on wet surfaces (reduced contrast), especially wet concrete.  Or if in a misty/rainy situation.

Curvier roads, will the code for excluding lines work correctly on tighter curves?

Words written on the roads.  This may create extraneous lines that confuse the line averaging algorithm.

### 3. Possible improvements

Instead of averaging over all lines, we could cluster the lines (in Hough space), then eliminate the resulting lines.

When merging lines from different edge detection methods, maybe use weighted-averages instead of just averaging.  This may be helpful if certain methods work better at various times (for instance, if one method is better during rain).

To smooth out the jitter, we could use a moving average for the lane lines, rather than just from a single image.  Using the current speed could determine how long of a moving average is worthwhile.

It would be nice to use ML for Hough parameter estimation.

