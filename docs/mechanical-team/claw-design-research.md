
# Tendon-Driven Method (Wire-Based System)

## Overview

The tendon-driven method is a robotic finger mechanism where motors remain in the arm, and wires (tendons) are used to pull and move the fingers.

- Inspired by human anatomy (muscles + tendons)
- Enables lightweight finger design → faster motion
- Provides safer interaction (impact absorption)
- Easier integration with vision systems / AI mapping

## Basic Structure of a Finger

A tendon-driven robotic finger consists of three main components:

1. Joints (Links)
Composed of 2–3 rigid segments
Connected using pins
Allow rotational motion
2. Wire (Tendon)
Pulling the wire → finger bends
Releasing the wire → finger extends
3. Motor + Pulley System
Located in the arm or base of the hand
Responsible for pulling the tendon

## Motion Principle
- Pull → Bend 
- Release → Extend
Extension Methods
- Spring-based return
    - Passive, automatic extension
- Second tendon (active extension)
    - Provides controlled extension

## Key Design Decisions
### 1. Number of Joints
- 2 joints : Simpler design, easier to build
- 3 joints : More natural movement, higher dexterity

Importance:

More joints → higher Degrees of Freedom (DOF)
Increased DOF → more precise manipulation
Trade-off: increased complexity and control difficulty

### 2. Number of Wires (Tendons)
Single Tendon (Underactuated System)
- One wire controls multiple joints
- Advantages:
    - Fewer actuators
    - Simpler and cheaper
    - Enables adaptive grasp
Dual Tendon System
- Two wires:
    - Flexion (bend)
    - Extension (straighten)
- Advantages:
    - Better control
    - Improved torque distribution
    - Higher stability
Key Insight:
    - Fewer wires → simpler system
    - More wires → more precise control

### 3. Wire Routing Path
Defines how the tendon travels through the finger structure

Importance
- Affects:
    - Joint movement sequence
    - Force distribution
    - Motion behavior
    - Stability of tension
    - Friction levels

- Types of Routing Methods
1. Pulley / Tube Routing
    - Tendon passes around pulleys near joints
    - Advantages:
        - Easy to design
    - Disadvantages:
        - Friction can be significant

2. U-Groove Routing
    - Tendon runs inside grooves built into the structure
    - Advantages:
        - Fewer components
        - Compact design
    - Disadvantages:
        - Harder to manufacture

3. Synchronous Routing
    - One tendon wraps around multiple joints
    - Forces joints to move together
    - Benefits:
        - One motor controls entire finger
        - Natural, adaptive grasp

4. Dual-Tendon Routing
    - Two tendons:
        - Flexor (bend)
        - Extensor (extend)
    - Benefits:
        - Precise control
        - Increased stability

5. Remote Routing
    - Motor located far (e.g., forearm)
    - Tendon transmits force over distance
    - Benefits:
        - Very lightweight fingers
    - Drawbacks:
        - Tendon stretching (elongation)
        - Increased friction

### 4. Wire Position and Torque
Concept: 
Torque depends on the distance between tendon and joint

Formula
 - Torque = Force × Distance

Key Insight
- Larger distance (moment arm) → higher torque
- Wire placement directly affects:
    - Rotation power
    - Joint control
    - Force efficiency

### 5. Friction in Tendon Systems
Problem
- High friction leads to:
    - Energy loss
    - Slow response
    - Poor control stability
Why It Matters
    - One of the biggest challenges in tendon-driven systems
Solutions
    - Use:
        - Pulleys
        - Bearings
        - Optimized routing paths

