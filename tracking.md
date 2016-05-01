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

<img src="{{site.baseurl}}/imgs/Stop.png" alt="Capture Window" class = "centered" width="500">

Initially, the position data and the time were saved in one file, but the sensor sometimes cannot pick up the locations of the facial features. This makes data processing harder since the data is not in order anymore if one face frame is missing. To fix this issue, we separate the position data for upper body joints and facial features in different file, and the time data is also saved in a different file. So, for one sign, three data files are: one for joint positions, one for facial feature positions, and one for time data. 

### 2. Data Stabilizing
To test the captured data, we import it to MATLAB. The origin is changed to the midpoint of the shoulders. For every frame, the midpoint is calculated and subtracted by the position of all other joints and facial points. We change the coordinate orientation as follow: the x axis grows backward, the y axis grows to right shoulder, and z axis grows up. The left and right eye positions are obtained by taking the average of the inner and outer corners of each eye. As the head does not move during a sign, the facial features are also static. Therefore, the average positions of the facial features are calculated across the frames, and only one set of facial points (right eye, left eye, nose, right mouth corner, and left mouth corner) are saved for reference. 

One way to check the captured data is to calculate the bone length. As we move our arms around, it is expected that the bone length stays the same. However, after calculating the arm length, we find that the length varies in an interval of 2 cm. Another way to see how the data fluctuates is to plot the raw data from the Kinect sensor. Below is the plot.

<img src="{{site.baseurl}}/imgs/graphKinect.png" alt="Kinect Graph" class="centered" width="400">

The bone length from joint a (with position (x<sub>a</sub>, y<sub>a</sub>, z<sub>a</sub>)) to joint b (with position (x<sub>b</sub>, y<sub>b</sub>, z<sub>b</sub>))  is calculated using the following formula. 

<img src="{{site.baseurl}}/imgs/f1.png" alt="Formula 1" class = "centered">

After calculating the bone length for the arm (shoulder to elbow), forearm (elbow to wrist), hand (wrist to palm), and hand tip (palm to tip), the average length for each segment is calculated. Then in each frame, a unit vector for each segment is calculated and multiplied to the average length of that segment. For example, the average length of the arms is larm. The position of the right shoulder is (x<sub>sh</sub>, y<sub>sh</sub>, z<sub>sh</sub>), the old position of the right elbow is (x<sub>el</sub>, y<sub>el</sub>, z<sub>el</sub>). The new position of the elbow is calculated as follow:

<img src="{{site.baseurl}}/imgs/f2.png" alt="Formula 2" class = "centered">

Where <img src="{{site.baseurl}}/imgs/unitVector.png" alt="Unit Vector"> is the unit vector from the shoulder to the old elbow position.

<img src="{{site.baseurl}}/imgs/f3.png" alt="Formula 3" class = "centered">

The new position of the wrist is calculated in the same manner with the forearm length l<sub>foreArm</sub>. The unit vector u<sub>wrEl</sub> from the old elbow position to the old wrist position is calculated. The new wrist position is:

<img src="{{site.baseurl}}/imgs/f4.png" alt="Formula 4" class = "centered">

The procedure is repeated for the hand and hand tip positions for all frames. After stabilizing the position data, it is seen that the motions plot for wrist and hand tip are smoother than the captured motion. Below is a comparison between the two.

<img src="Kinect Graph" class="centered" width="400">

<img src="{{site.baseurl}}/imgs/graph_stabilized.png" alt="Stabilized graph" class="centered" width="400">

The position data is exported in to *.mat files, one for the body joints, one for the facial features, and one for the time.

<img src="{{site.baseurl}}/imgs/Dad.png" alt="Dad" class="centered" width="400">

<img src="{{site.baseurl}}/imgs/Mom.png" alt="Mom" class="centered" width="400">

### 3. Mapping 
To map the position data to the robot, the origin is also chosen to be at the midpoint of the robot shoulders. The axis orientation is also the same as in data stabilization. The hand position is prioritized since its position also determines the meaning of a sign. For example, one of the signs below means “mom,” and the other means “dad.” The sign with the thumb placed at the chin means “mom”, and the sign with the thumb placed on the forehead means “dad.”

To determine the hand position, the coordinates is divided into two parts: y≤0, and y>0. If the y and z components of the hand is positive, the distance (yz plane) from the hand position to each of the facial points is calculated for the stabilized data set. The robot’s hand position on the yz plane is calculated using the obtained distance from previous part. The x component of the robot hand position is the same as the x component of the stabilized hand data.  However, if the y and z components of the robot hand is negative, its position is calculated based on the relative distance to both shoulders. The x component of the robot hand is the sum of the x component of the stabilized data and half of the robot’s body thickness. With the hand position found, the wrist position is calculated using the hand to wrist unit vector (stabilized data) and the robot hand length.
The positions of the shoulders throughout a sign is fixed because it is observed that the shoulder position do not change in a sign. The unit vector from shoulder to elbow u<sub>elSh</sub> in the stabilized data set is calculated and used to calculate the temporary position of the elbow.

<img src="{{site.baseurl}}/imgs/f5.png" alt="Formula 5" class = "centered">

With three points (wrist, elbow, and shoulder), a plane equation is formed. With the position of robot’s wrist being (x<sub>wrR</sub>, y<sub>wrR</sub>, z<sub>wrR</sub>), the plane equation can be formed as follow.

<img src="{{site.baseurl}}/imgs/f6.png" alt="Formula 6">

Using equations (1), (2), and (3), the true position of the robot elbow is found.  
This process is repeated for every frame, and the position for each joint of the robot is calculated. Below is the comparison of the captured, stabilized and mapped motion for the sign “hello.”

<img src="Kinect Graph" class="centered" width="400">

<img src="stabilized graph" class="centered" width="400">

<img src="{{site.baseurl}}/imgs/graph_Mapped.png" alt="Mapped graph" class="centered" width="400">


