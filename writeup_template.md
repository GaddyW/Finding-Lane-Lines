# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/edgessolidYellowLeft.jpg "CannyMask"
[image2]: ./test_images_output/linessolidYellowLeft.jpg "Hough"
[image3]: ./test_images_output/solidYellowLeft.jpg "SolidMarker"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consists of the following steps:
1)  Grayscale the image.  This is necessary to simplify the gradient finding used in Canny.  Better to check one image instead of 3 color schemes.

2)  Use Guassian Blur to get rid of noisy elements that would trip up Canny

3)  Canny, to identify points of significant color transition in the image that could identify a lane marker.

4)  Masking - We're only interested in the region of the image that could be a lane marker.  So, to simplify our work and to reduce the potential number of irrelevant line segments that would be identified by the Hough transform, we focus only on the symmetric trapezoid directly in front of the camera. (In the discussion below, I'll reflect on the shortcomings of this method.  But for now, with straight road segments, it works).
![alt text][CannyMask]

5)  Hough transform to find lines from the point cloud we've created in the previous steps.
![alt text][Hough]

6)  Assign the lines from the Hough transform to either the right or the left lane markers.  Rather than rewrite draw_lines(), I simply added the code to the pipeline.  You will find it in the notebook directly after the Hough transform.  If a line has negative slope and its points are to the left of the polygon mask's midpoint, it is assigned to the left lane marker.  If a line has positive slope and its points are to the right of the polygon's midpoint, it is assigned to the right lane marker.  

7) Draw a line the full length of the lane marker.  To do this, I decompose all of the lines associated with each lane marker to their constituent points (These are stored in the lists left_x[], left_y[], right_x[], right_y[]).  I then run a 1st order linear regression on the points using polyfit to calculate a line of function y=mx+b for the left and right lane markers.  I then draw these from their y-intercepts on the original image.
![alt text][SolidMarker]



### 2. Identify potential shortcomings with your current pipeline

1) While I understand the parameters of the Hough and Canny operations, I don't have the intution that comes from experience about what would be the best parameter set.  It  would be good now to learn how experts tweak these settings.

2) The polygon mask makes sense for a straight lane, but it is useless in a curve as can be seen pretty easily from running the challenge video.  I also think it may lose its value if the drivers swerves in the lane.  It seems to work best when driving in the middle of a perfectly straight lane.

3) I don't love the algorithm I used for assigning lines from the output of the Hough transform to left and right lane markers.  While it works for these two example videos, I don't think it generalizes well to all driving conditions.  For example driving around a curve, or dealng with legitimate road signs that are painted on to the lane.  Slope and position in relation to a fixed polygon alone are not enough.  I'll discuss possible improvement below.  


### 3. Suggest possible improvements to your pipeline

For improving the Hough and Canny operations:  At the most basic level, it would be good to learn from a seasoned pro.  At a more advanced level, I have a feeling that these parameters should be dynamically adjusted based on lighting, weather, and road conditions.  It might be possible to learn these parameters with machine learning and then to update dynamically.

For assigning line segments to left and right lane markers - as I mentioend, I used slope and relation to the midpoint of the polygon mask. 
    - At the least, I would add color.  If we are lucky enought to have yellow and white lane markings, then color could help differentiate. 
    - Also, I would filter out those line segments with slope near 0.  Perfectly horizontal lines from things like the car's hood, changes in road surface, or small elements on the road that make it through the Hough transform could be filtered out this waythe polygon mask - 

For extrapolating the line - I would move up to a second or third order polynomial regression to be able to account for curvature in the lane.

For the polygon mask - I'm still wrestling with this one.  At the least, our mask should start at the bottom of the image around our camera.  The question is, how to move it up the image.  A slowly contracting polygon doesn't work for every driving condition.  
Another potential improvement could be to ...
