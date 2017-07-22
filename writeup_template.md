# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### Pipeline

My pipeline consists of 12 steps:

Change colorspace to HSL. Originally I just used the RGB colorspace and things worked well for the first two problems but failed on the challenge. I added some screen shots from the challenge to my test images and identied that the main failure was detecting the yellow line on the gray section of the road. I eventually tried adding a color mask after looking at all the OpenCV methods that the project suggested. Searching for information on the cv2.inRange() method resulted in an interesting article on using a color mask for object detection

Mask image to focus on yellow and white regions

Change colorspace to Grayscale for effective edge detection

Gaussian blur image to eliminate noise that could be incorrectly identified as important edges

Run Canny edge detection

Mask a region of interest to isolate edges found on the road in-front of the car

Run a hough-transform to identify contiguous lines discovered using Canny edge detection

Split the lines into left and right lane candidates by paritioning on the gradient. negative gradients are on the left, positive are on the right. The lines on the left would seem to have a positive slope, but since the y axis is inverted the sign of the gradients is inverted too

Average the identified lines, weighting the average by the lengths of the lines and the similarity to the lines from the previous frame

Perform a final check to make sure the new lanes haven't devitaed from the lanes in the previous frame too much. If they have notify the user of a possible error. 

Average the new line to be slightly closer to the previous line to smooth the line transitions between each frame

Draw the lines onto the image or frame for display



![alt text][image1]


### 2. Possible Issues


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Improvements

A possible improvement would be to ...

Another potential improvement could be to ...
