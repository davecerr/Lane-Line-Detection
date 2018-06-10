# **Finding Lane Lines on the Road**

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. First, I converted the images to grayscale, then I perform a Gaussian blur to remove noise that may cause problems for Canny edge detection.

Then I perform Canny edge detection. There is some finesse required in choosing the threshold parameters such that we pick up the desired lines.

After this we are still picking up lines outside of the road e.g. bridges, landscape etc. We perform a region mask that sets all pixels outside a region of interest (the road) to zero. By using a bitwise_and operation to combine the region mask with the result of Canny edge detection we now have an image that only identifies lines on the road.

However, since the Canny edge detection is picking up pixels with large gradients, it is only highlighting pixels - these can be considered as candidate points for lane lines. We want to highlight lines however. Individual pixels are transformed to sinusoidal curves in polar Hough space. Each point on the curve represents a potential line that could be drawn through the original iamge pixel. Intersections of curves in Hough space represent lines that can be drawn on the original image that pass through multiple points. Therefore to identify lane lines, we will perform a Hough transform and only keep points in Hough space with a certain threshold of curve intersections. These represent lines on the original image passing through a certain threshold number of candidate points and thus represent our best guess of the lane line locations. I have written a blog post explaining how this works - see https://daviderrington.net/2018/06/05/the-journey-begins/

![][result_img.png]

At this point we are able to overlay these lines on the original image. However, the results are quite unstable and I can envisage car salesmen and saleswoman of the future having considerable difficulty persuading any customer to board, let alone buy, such a vehicle. This problem is that we want to draw a single line on the left and a single line on the right. We also want the lane line to extend from the bottom fo the screen to the horizon. This is addressed by modifying the draw_lines() function. I do this by  creating a list of positive gradient lines and negative gradient lines from the first video and then for each list I take a weighted average of their gradients and endpoints (longer lines get a higher weighting). At the end of all this we produce a single positive gradient line (left lane) and a single negative gradient line (right lane). These can then be plotted with greater thickness for visual effect.

Ultimately this is applied to video footage (a collection of images) and by overlaying the lane line estimate on each frame, we get good results for lane line detection.


### 2. Identify potential shortcomings with your current pipeline


Our results are excellent for straight line driving. One potential shortcoming would be what would happen when when we try to turn a corner. This is shown in model's inability to deal with the challenge video.

Another shortcoming could be if we hit a frame in the video where the Hough transform is unable to identify any lines. This actually causes my model to be break when evaluating the challenge video.

I have some ideas on hwo to address these that I will outline below.


### 3. Suggest possible improvements to your pipeline

Regarding the cornering issue, when my model tries to evaluate the challenge video, the lane lines essentially appear as a giant X across the screen. This is because we are right at the apex of the bend and the longest straight line it can find is "around the corner". To deal with this we could modify our draw_lines() function to check the range of positive and negative gradients that have been found. If the variance is small, we can be confident we are on a straight stretch of road and we can just proceed to draw our lines using the pipeline above. However, if the variance is large (above some tolerance), we can be confident we are on a corner. In this case, we might want to try and fit some polynomial curve to the points on the bend identified by Canny edge detection.

To deal with the pipeline breaking when it hits a frame for which it cannot identify any lines, we could introduce a queue that stores the last line that was succesfully identified. That way, if we hit such a frame, we can just use the previously stored gradient.
