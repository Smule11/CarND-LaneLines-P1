# **Finding Lane Lines on the Road** 

## Writeup Template

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/all_detected_lines.png "All detected lines"
[image2]: ./examples/lane_line_detection_gone_wrong "Inadequate lane line detection"


---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of the 14 steps outlined below. 
I also shortlisted five key variables:

The Canny edge `low` and `high thresholds`. And the Hough transform `min_line_length`, `threshold` indicating the minimum number of intersections in Hough space representing a line in image space, and `max_line_gap` relating to the maximum gap in pixels between connectable line segments. Tweaking these had the most immediate and straightforward impact on lane line detection.

- First, I converted the images to grayscale
- Then I applied the gaussian blur
- Then I used `cv2.Canny` to locate edges based on the pixel gradient changes.
- Then I applied a mask over the whole image to only inspect the edges where I think the lanes will be.
- Then I used a Hough Transform to find straight distinct lines.

I modified the draw_lines() function to filter on the gradient, and then store each acceptable line into either 
positive or negative (i.e. right or left lane) lines:

- I stored the lines with gradients that were between about 30-55 degrees into appropriate arrays.

Sometimes the image is such that the lane lines don't stand out much from the tarmac.
Here I created a sub-if-clause just in case the main code fails to detect the lane lines in a frame of the video.
This sub clause is more lenient and catches more subtle lane lines, but may be less precise too as it may collect 
extra unwanted lines.
This keeps the code running when the main code fails.

- I created an if clause which mirrors the steps above but with different constants.
- The other difference was that in the if clause the image selects for pixels with a minimum yellowness:
```
        # define range of yellow color in HSV
        lower_yellow = np.array([190,170,0])
        upper_yellow = np.array([255,255,255])
        # Threshold the image to get pixels with a minimum red/green level
        mask = cv2.inRange(image, lower_yellow, upper_yellow)

        #plt.imshow(mask)
        res = cv2.bitwise_and(image,image, mask= mask)
```
- After the end points of lines which are in the expected area and with an expected length and direction are found,
they are stored in arrays.
- The gradients of the lines pointing up and left are averaged, and so too for the other set of lines. 
- Likewise the average position of these end points are found for the left and right lane markings.
- Then usin `cv2.line` and using the calculated x-intercept, extrapolated lines with equivalent average gradients 
are superimposed onto the line_image array.
- The line_image array and original image are then combined and set to the _result_.
- Finally, a count is returned, counting the number of times the intermediate if-clause is accessed.


Here is the code only paying attention to the blue-detected lines which are angled in the directions 30-55 degrees:

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be in being too sensitive and detecting lines which weren't lane lines.
Vice versa, another problem would be not having the canny_edge sensitive enough to detect lane lines.

Both of these can often be resolved through tweaking the settings.

Another shortcoming would be if the lanes were outside the masked area then the program would fail.
In my case the program would also fail to detect the lanes if the lane lines did not angle across the image at 
the expected directions/angles.

These can be resolved through not masking, or restricting the line detecting system, but then it highly increases 
the chance for mistakenly detecting lines as lane lines.

Here the code was set-up such that it was unable to detect the yellow line:

![alt text][image2]


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to apply a filter over the image that differentiates the lane line colouring more.
For example only looking for yellow and white pixels. 

Another potential improvement could be to have a bright torch/headlight to reduce the affect of shadows on the 
road ahead. 

Also implementing code to tweak the key variables in real time to maximise expected lane line detection.
