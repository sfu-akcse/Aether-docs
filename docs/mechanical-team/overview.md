# Mechanical Team Overview

### Core Objective

To design and build a robotic arm that can mirror human movements and perform grabbing tasks

#### Technical milestones

Mechanical & Integration

Focus: Physical design and the software-to-hardware bridge.

- Physical Design: Design the robot arm, including mounts and structural components, optimizing for torque and Degrees of Freedom (DoF).

- Firmware & Connectivity: Configure Raspberry Pi as a socket client to receive software signals and link them to physical mechanical movement.

- Motor Control: Interface Raspberry Pi with a servo controller to manage motors (Servo and Stepper).

- Actuation Logic Example: A "fist" gesture triggers the algorithm to send a signal to the Raspberry Pi; the Pi processes this through application software to trigger the grabbing servo.
