## Project: Search and Sample Return
In this project I coded the brains of this rover to map the area and pick up rocks:
<img src="RoverCollector-x10.gif" width="960" />

---

### Here below is the full video (YouTube) at 1x speed.  The above animation is 10x speed.

<a href="https://www.youtube.com/embed/m-aD7KrcaeE" target="_blank">
  <img src="http://img.youtube.com/vi/m-aD7KrcaeE/0.jpg" alt="rover driving on mars-like area collecting yellow rocks" width="240" height="180" style="text-align:center;border: 10px solid red" />
</a>


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

[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  


### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
As described above, I used thresholding to determine the navigable terrain, and HSV colorspace threshold + OpenCV contour detector to find the rocks.  The below video was generated from my own training data collected in the simulator and processed in the notebook (locally)

<img src="notebookoutput.gif" width="960" />

The animated image at the top of this document shows the results when run in autonomous mode.


### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.


#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

## Based on the provided sample code, I made the following additional changes to complete the project

A. Perception

1. Get the drivable/nondrivable areas
- Use thresholding to detect bright areas as drivable
- perspective transform the image to top view
- use the zeros of the transformed image for a null mask
- create obstacle map by subtracting the mask from the inverse of the drivable image
- cleared the top section of the top view image to ignore highly distorted areas 
- transform to world and add to map with incremental addition/subtraction of detected terrain

2. Get visible rock locations
- Convert the HSV and choose matching hue for thresholding the yellow rocks
- Detect the bottom pixel of the rock and draw small circle on empty frame
- Transform perspective to top view, and detect lowest pixel as rock coordinate
- Draw a circle on the topview map in the green channel
- transform to world and add to map
- set a flag if rock is visible, for decision step

3. Calculate navigation angles
- add heat map of visited areas and ignore pixels recently visited (ie favor angles of new cold pixels) 
- convert drivable image pixels to polar coordinates
- convert visible rock locations to polar and add multiple copies to drown out the regular angles

B. Decision
1. Add a stuck state by keeping moving average of rover velocity. Turn if stuck
2. Add a bias toward right by adding 25% of standard deviation of angles(100% would mean follow right edge of drivable terrain)
3. Add checks to slow down but keep moving if rock is visible (visibility set in perception step)
4. Add brake when near sample (built in info) to allow pickup to be sent

C. Hyperparameters
1. Throttle set to .3 (.1 when rock is visible)
2. Max velocity 2 m/s
3. Don't update world map if pitch or roll > 2 degrees
4. Heatmap set to minimal influence, essentially unused.

D. System
1. Windows
2. 1920x1080 at Fantastic
3. Approx 30 FPS

E. Results
1. 85% mapped in 10 minutes 
2. 75% Fidelity
4. Able to detect stuck state and get unstuck
5. All 6 rocks detected
6. All 6 rocks collected
7. Rocks returned to middle (but rover not programmed to stop)

F. Failures and Future
1. Sometimes rover goes in circles in wide open spaces due to clamping (add visited places)
2. Sometimes rover runs into small rocks directly in front (add small obstacle detection)
3. Sometimes rover returns to explored areas before finishing mapping (add visited places)



