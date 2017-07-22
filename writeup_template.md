# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[//]: # (Image References)

[image1]: ./test_image_output/final_pipeline.jpg "Final Pipeline Results"
[image2]: ./test_image_output/line_histograms.jpg "Line distributions"

---

### Reflection

### Pipeline

My final pipeline consists of 13 steps:

* Change colorspace to HSL.
* Mask image to focus on yellow and white regions
* Change colorspace to Grayscale for effective edge detection
* Gaussian blur image to eliminate noise that could be incorrectly identified as important edges
* Run Canny edge detection
* Mask a region of interest to isolate edges found on the road in-front of the car
* Run a hough-transform to identify contiguous lines discovered using Canny edge detection
* Split the lines into left and right lane candidates by paritioning on the gradient. negative gradients are on the left, positive are on the right. The lines on the left would seem to have a positive slope, but since the y axis is inverted the sign of the gradients is inverted too
* Average the identified lines, weighting the average by the lengths of the lines and the similarity to the lines from the previous frame
* Extrapolate the average line so that it has the same gradient and intercept but extends to the required y values below the horizon and at the bottom of the frame.
* Perform a final check to make sure the new lanes haven't deviated from the lanes in the previous frame too much. If they have notify the user of a possible error. 
* Average the new line to be slightly closer to the previous line to smooth the line transitions between each frame
* Draw the lines onto the image or frame for display

Originally I setup a pipeline very similar to the one covered in the lectures (grayscale->blur->edge->mask->hough->average->extrapolate). This worked well for the first two problems but failed on the challenge. The main issue was the section of gray road with the yellow line on the left. The pipeline seemed unable to differentiate the two at all. Since I could clearly see the yellow line I researched into the differences between human and machine perception and found out about the HSV and HSL color spaces used in machine vision to better approximate human perception of color. I tried mapping to these colorspaces but any differentiation seemed to be lost when I converted to grayscale for the edge detection. The pipeline was actually worse.

On recomendation from my mentor I tried isolating a single channel after the HSV color conversion but this still didn't improve my solution. I revisted the suggested OpenCV methods in the project and the first, inRange, yielded an interesting result. On searching for more details of the function I found this article: http://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_colorspaces/py_colorspaces.html

In the article they also convert to the HSV space first, then apply a color mask to isolate an object for object detection. I tried using the mask in the RGB colorspace for yellow and white and my pipeline could now successfully pick out the yellow lines on the gray section of road, but now it couldn't see the yellow line covered in thick shade. I tried creating a mask in the HSV space and this successfully picked out the yellow in all examples, but now the white lines were failing to be detected. Looking at the color cylinder for HSV it's actually quite hard to see how to define the color white. The HSL model however makes it very easy to identify all forms of the white color with a variable sensitivty. I was able to detect the lines in all situations using a mapping to the HSL colorspace (actually H*L*S in OpenCV) then masking the colors of interest and continuing with the rest of my pipeline. I see now how the HSL color space is better suited to masking since you can define a range that covers a particular Hue, or color, for a large range of lightness and saturation which covers scenarios like this where a color is heavily in shade.

After obtaining a good set of lines found in the image I had to determine how to reduce the set to a single left and right lane line. The project was pretty clear that I should average the lines but I actually used a quick hack where I just selected the longest line as the correct line, this worked pretty well most of the time. I assumed averaging would work better and be more robust so I started examing how best to do this. I plotted a historgram to get a sense of the distribution of gradients and y-intercepts. Most of them were in a region that aligned with the actuall lanes, but there were still a number of outliers and I knew these would throw my average off. I eliminated any verticle lines, as well as any with an absolute gradient less than 0.5 as the true lane lines were always above this. I still had a few outliers however and I didn't want to be too rigid in the kinds of gradients I would accept. Knowing how well my single longest line selection had worked I tried plotting the histogram of line gradients and intercepts again, but this time I used the line lengths as weights in the histogram. This gave me a much better result, my outliers were greatly diminished and the true lines were amplified. I decided to use a weighted average with the line lengths as weights.

![alt text][image2]

Before I discovered the color space conversion and color masking I tried a different approach where I used the lane lines from the previous frame in the video examples. With my new pipeline I didn't need this technique anymore, but I left it in anyway as it actually smooths out the visualisation of the lane lines quite nicely. I use the previous lane lines to influence the averaging of all the calculated hough lines by adding it to my line length weights. I also use the previous lane lines to shift the newly forecast lines slightly towards the lines from the previous frame for a smoother rendering. I'm also able to calculate the difference between the current and previous lines and alert the user if this difference passes a threshold. A sudden jump in lane lines between frames is a good indication that something somewhere has gone wrong. It also provides the option to fall back to a previous lane line if no good lanes can be found in the current frame. This is a little dangerous however and probably something that could only be used for a frame or two. I've turned this feature off since I was able to find good lines in all frames of the provided videos. Note that each of the video examples has a line ```white_clip = clip1.fx(process_image_wrapper, previous_lines, True, False)``` The second last argument determines if historical data is used in the calculation, the last determines if debugging is enabled. Debugging will render the candidate lines on top of the video as well as the detected lane lines. Debugging will also save a copy of a video frame where the variation between previous and current lane lines indicates a possible error.

Below is the result of the final pipeline on all of the example images. In the left column is the original image with the lane lines in red and the candidate lines in green overlayed on top. The middle column is the color space converted and color masked image. The final column is the image after edge detection and masking the region of interest.

![alt text][image1]


### 2. Possible Issues


* Only accepting lines with a gradient above 0.5 might be too rigid in some situations, I would need more example videos to confirm this.
* Influencing current lines by using the previous lines might not be ideal. If the car hit a hole and shifted orientation quickly the previous lines would be poor indicators of where the new lane lines are. I'm not sure how likely this is though and it would still converage to the new truth quickly
* The region of interest masking is based on a hardcoded horizon, in reality the horizon would change in different scenarios, e.g. the car climbing a hill
* This model uses straight lines and doesn't give a clear indication of the required steering angle when driving around a corner


### 3. Improvements

* Automated horizon detection
* Non linear model for understanding turns better
* SLAM to navigate using landmarks as well as lane lines, for scenarios where there are no clear lane lines.
