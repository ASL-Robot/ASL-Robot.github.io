---
title: Tracking and Mapping
layout: post
permalink: /tracking.html
---
The learning phase of the project is completed with motion capture, data stabilization, and position mapping. The mapped position data is tested in robot simulation first. After testing, the position data will be applied onto the robot.

### 1. Tracking
The motion of the upper body joints is tracked and recorded using an RGB D sensor. The sensor that is used is the Kinect Sensor for Xbox One, and the code is written in C#. This sensor is capable of tracking 27 joints of a human’s body (Microsoft). For this project, we only need the data for the shoulders, elbows, wrists, hands, and hand tips. The sensor is also able to track 1347 points on a human’s face, but we only record the positions for the eye corners, nose, and mouth corners. The position data is with respect to the sensor (the sensor’s position is at (0, 0, 0)). From the Microsoft's website, the Kinect's coordinate orientation is defined as follow:

<img src="{{site.baseurl}}/imgs/KinectCoord.png" alt="Kinect's Coordinate System" class = "centered" width="500">

Below is the algorithm that we used to capture the position data.

![Algorithm Flowchart]({{site.baseurl}}/imgs/Algo.png)

The body joint tracking sample code is provided in the Kinect SDK. However, since the position data of the facial features is also important, we modified the sample code to track the facial points. Because only data during a sign is needed, we also add “start capture” and “stop capture” buttons to make sure that only valid data are captured. The recording rate is 30 fps. At every frame, the positions of the upper body joints and facial feature are recorded. The time for each frame is also saved to the data file, and time 0 second is when capture starts. Below is what the capture window looks like.

<img src="{{site.baseurl}}/imgs/Stop.png" alt="Capture Window" class = "centered"  width="500" >

Initially, the position data and the time were saved in one file, but the sensor sometimes cannot pick up the locations of the facial features. This makes data processing harder since the data is not in order anymore if one face frame is missing. To fix this issue, we separate the position data for upper body joints and facial features in different file, and the time data is also saved in a different file. So, for one sign, three data files are: one for joint positions, one for facial feature positions, and one for time data. 

### 2. Data Stabilizing
To test the captured data, we import it to MATLAB. The origin is changed to the midpoint of the shoulders. For every frame, the midpoint is calculated and subtracted by the position of all other joints and facial points. We change the coordinate orientation as follow: the x axis grows backward, the y axis grows to right shoulder, and z axis grows up. The left and right eye positions are obtained by taking the average of the inner and outer corners of each eye. As the head does not move during a sign, the facial features are also static. Therefore, the average positions of the facial features are calculated across the frames, and only one set of facial points (right eye, left eye, nose, right mouth corner, and left mouth corner) are saved for reference. 

One way to check the captured data is to calculate the bone length. As we move our arms around, it is expected that the bone length stays the same. However, after calculating the arm length, we find that the length varies in an interval of 2 cm. Another way to see how the data fluctuates is to plot the raw data from the Kinect sensor. Below is the plot.

<img src="{{site.baseurl}}/imgs/Stop.png" alt="Capture Window" class = "centered" width="500">

The bone length from joint a (with position (xa, ya, za)) to joint b (with position (xb, yb, zb))  is calculated using the following formula. 

<img src="{{site.baseurl}}/imgs/f1.png" alt="Formula 1" class = "centered">


### 3. Mapping 
