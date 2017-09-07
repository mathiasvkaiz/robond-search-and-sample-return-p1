## Project: Search and Sample Return
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./misc/color_thresholdin.png
[image2]: ./misc/color_thresholdin_2.png
[image3]: ./misc/rock_detection.png
[image4]: ./misc/rock_detection_2.png
[image5]: ./misc/process_image.png
[image6]: ./misc/map.png
[image7]: ./misc/worldmap.png
[image8]: ./misc/decision.png
[image9]: ./misc/decisiontree.png
[image10]: ./misc/source_destination.png
[image11]: ./misc/rover_vision.png
[image12]: ./misc/angles.png
[image13]: ./misc/rock_detection_3.png



## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it and here is a link to my [project code](https://github.com/mathiasvkaiz/robond-search-and-sample-return-p1/blob/master/code/Search_And_Sample_Return.ipynb)

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
As the obstacles on the map are dark areas (and this can be said for all three color channels RGB) and navigatebale area instead is quite bright we can separate obstacles and navigatable space by definined a color threshold.

![alt text][image1]
At this code snippet in the notebook we identify the obstacles by defining a threshold border with rgb_thresh parameter. Afterthat we analyze all three RGB channels separately and check if the given pixel value is above or below the threshold. If this is the case we set a 1 as value in the array (that has the same size as the image) or an 0 if it is below the threshold (done on array initialization -> color_select = np.zeros_like(img[:,:,0]))

![alt text][image2]
Here the defined function "color_thresh" is called and the values for definig obstacles are set.


For rock identification a new function is added
![alt text][image3]

In this function we also have defined a threshold for each RGB channel "levels=(110,110,50)".
The we compare eah RGB cnalle value and defined 1 or 0 based on the logic
pixel value greater than R channel threshold, greater than G-channel threshold and smaller than B-channel threshold.


Following images shows the calling of the rock detection and mapping to the world map
![alt text][image4]


#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 

![alt text][image5]

In this method all relevant functions are applied. 

First we use **perspect_transform** to switch the robot image to an above image. We threshold the returend warped array. After that we create the obtsalce map based on the thresholded navigatable values. In this step we also use the second return array of **perspect_transform** (mask) to be multiplied with the threshed navigatable values. So the 0 and 1 of the mask (defining the visible range) are regarded. As a result we get 1 everywhere we have obstacles.

After that we convert to rover coordinates **rover_coords**.
With that information and the given worldmap we build the world based on coords, position, yaw and map with **pix_to_world** function.
This step is done for navigatable space and obstacle space
![alt text][image6]

After that we are able to update the worldmap. Based on world and obstacle world channels we update the blue channel wehre we can navigate to and the red channel where obstacles are placed.
![alt text][image7]

Here we could get some confilcts between navigatable cpace and obstacle space so that is why a "normalization" is used. Navigatable space beats obstacle space by setting the red channel to 0 if we have a value of blue channel (navigatable space) greater than 0.

After that the area  "rock_map = find_rocks(warped, levels=(110, 110, 50))" defines where to map rocks on the worldmap in case we have found a rock.





### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

**decision_step** function
![alt text][image8]

is based on following decision tree
![alt text][image9]


**perception_step** function
Is based on the **porcess_image** function out of the notebook. It takes over nearly all steps with some slightly differences that are explained now.


Here the source area and destination array is defined so that we can perform the perspective transformation.
![alt text][image10]

Here the rover vision image is updated
![alt text][image11]

In general all position and yaw values are from Rover class now.
Also the worldmap is updated in Rover class as well.
To favour navigatable terrain we changed the Red And Blue Channel update to

Rover.worldmap[y_world, x_world, 2] += 10
Rover.worldmap[obs_y_world, obs_x_world, 0] += 1

So we add 10 each time we discover navigatable terrain and 1 each time we detect and obstacle.



Also we now calculate the polar coordinates and distance of the pixel.
![alt text][image12]
Then we update the Rovers angle so that the **decision_step** function can work with that information

For the rock detection step more code is added as the rocks also have a vertical space. So get the minimal distance to rock and define it as center point.
![alt text][image13]









#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

I used following settings:
1024x768
Good
Windowed

My apporach is to seperate between navigatable and non navigatable space and transforming the image from the camera to a top down view. Based on info of navigatable space and the top down view i can build a map of all image frames coming in from the rover cameras and compare it with the world map.

Following techniques were used:
- Image perspective transformation given by openCV framework
- Color thresholding for seperating navigatable and non navigatable space on al RGB channels
- Coordination processing (e.g. Polar coordinates and distance)
- Angle adjustment
- Mapping of coordinates to map
- Coloring the map in different colors (Red and Blue) for navigatable and non navigatable space
- Simple decision tree for decision making

Checking the autonomous mode i can see that the rover not always behaves like wanted. Sometimes it gets stuck in an obstacle or cannot proceed navigating on free terrain (driving around in circles can happen).
I would say that a very good starting point for improvement is a much more sophisticated hanlding of obstacle and free space overlaps. In this simple version i have just removed all obstacle values in case a free space value is also given. This could be handled much better in future. 

We have to consider that not is not alwys possible to have a clear difference in pixel values for navigatable and non navigatable space. Here we could also use a much more sophisticated apporach by using filters and so on.

For decision making a much complicated decision tree could also avoid bugs in autonomous movement. Or even other techniques like Deep Learning (as in the Self-Driving Car Course) could be applied as well.

So we have much space for improvements in future.





