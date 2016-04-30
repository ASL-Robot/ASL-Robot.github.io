---
title: Tracking and Mapping
permalink: /tracking.html
---
# Tracking and Mapping
The learning phase of the project is completed with motion capture, data stabilization, and position mapping. The mapped position data is tested in robot simulation first. After testing, the position data will be applied onto the robot.

### 1. Tracking
The motion of the upper body joints is tracked and recorded using an RGB D sensor. The sensor that is used is the Kinect Sensor for Xbox One, and the code is written in C#. This sensor is capable of tracking 27 joints of a human’s body (Microsoft). For this project, we only need the data for the shoulders, elbows, wrists, hands, and hand tips. The sensor is also able to track 1347 points on a human’s face, but we only record the positions for the eye corners, nose, and mouth corners. The position data is with respect to the sensor (the sensor’s position is at (0, 0, 0)). Below is the algorithm that we used to capture the position data.

![Algorithm Flowchart]({{site.baseurl}}/imgs/Algo.png)

The body joint tracking sample code is provided in the Kinect SDK. However, since the position data of the facial feature is also important, we modified the sample code to track the facial points. Only data during a sign is needed, we also add “start capture” and “stop capture” buttons to make sure that only valid data are captured. The recording rate is 30 fps. At every frame, the positions of the upper body joints and facial feature are recorded. The time for each frame is also saved to the data file, and time 0 second is when capture starts.

### 2. Data Stabilizing

### 3. Mapping 
