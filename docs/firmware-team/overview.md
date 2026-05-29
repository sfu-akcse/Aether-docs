# Firmware Team Overview

The Firmware Team will mainly work on these roles:

1. Socket Communication with Software
   - Configure the Raspberry Pi for real-time communication with the host computer through ROS2 and TCP sockets

2. Kinematics Calculations
   - Parse continuous spatial coordinates (X, Y, Z) provided by the MediaPipe vision pipeline into valid joint-space targets.
   - Calculate the specific degrees of rotation required for each joint to match the observed human hand poses.
   - Implement mathematical constraints to ensure calculated angles do not exceed the physical limits of the robotic joints.

3. Motor Control (Robot Movement)
   - Establish and manage the physical hardware connections (GPIO, Serial/UART, or I2C) between the Raspberry Pi and the servo motors.
   - Translate the calculated kinematic angles into precise electrical control signals to actuate the motors smoothly.
   - Read physical encoder data to verify the motors have reached their target positions and report this feedback up the ROS 2 chain.

4. Manual Arm Control
   - Wire and configure physical potentiometers to allow direct, manual manipulation of the arm without reliance on the host PC's vision software.
   - Process analog voltage readings (via an external ADC) and map them to physical motor angles.