# Software Team Overview

Software Team will mainly work on these roles:

1. Computer Vision (Hand Motion)
   - Trace the hand motion using the open-source, pre-trained framework with 3D coordinates in real-time using a standard webcamera from the laptop.
   - Based on the 3D coordinate that we are receiving from the camera, the system will return XYZ coordinate using Inverse Kinematics.

2. Machine Learning (May use or not)
   - Faster method is to simply calculate the mathematical distance between the thumb tip and index finger tip. If the distance drops below a certain threshold, trigger the "grab" command.
   - May train the Machine Learning model to recognize a grab (closed hand) or open hand.

3. Robot Arm Simulation (ROS2 + Gazebo)
   - The Computer Vision part will act as a ROS 2 Node and publish the desired XYZ coordinate. A ROS 2 control package will calculate the Inverse Kinematics and move the simulated arm in Gazebo.

4. Socket Communication with Physical Robot Arm
   - Once the physical Robot Arm is ready by the mechanical team, the system will be able to communicate over the TCP socket to arm's controller and controll the coordinate that we want.

The flow will be:

Webcamera => Computer Vision for hand motion => ROS 2 => Inverse Kinematics Calculation => Gazebo (Simulation) / Socket (Physical Robot Arm)
