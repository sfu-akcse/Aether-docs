# Robot Arm Torque Analysis

This document provides the torque analysis for the robot arm design related to issue #52.

## Summary
This work includes:
- wrist torque estimation
- elbow torque estimation
- shoulder torque estimation
- gripping torque estimation
- an interactive HTML dashboard for live parameter adjustment

## Interactive Dashboard
You can access the dashboard here:

[Open the robot arm torque dashboard](/Aether-docs/robot-arm-torque-dashboard/)

## Notes
The dashboard is based on:
- payload fixed at 300 g
- ST3215 motor mass = 0.055 kg
- wrist motors = 3
- elbow motors = 1
- shoulder motors = 2
- rated torque per ST3215 = 10 kg·cm