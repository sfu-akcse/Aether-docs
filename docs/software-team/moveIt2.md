# MoveIt 2 Documentation

## Overview
MoveIt 2 is an open-source robotic manipulation framework built on ROS 2. It provides a complete set of tools for controlling robotic arms and manipulators, including motion planning, kinematics, perception, and control. It helps robots plan and safely execute movements using ROS 2 communication.

## Core Features

### 1. Motion Planning: 
Motion planning is how the robot decides which way to move from Point A to Point B without hitting its own base or the table it’s sitting on.

- Grasp Planning: Deciding where to place the fingers so the object doesn't slip.
- Pick and Place: A full workflow—move to object, open gripper, grasp, lift, move to destination, and release.

**Types of constraints (rules you can add):**

- Position constraint
- Orientation constraint
- Visibility constraint → must be visible to a sensor
- Joint constraint → joint angles must stay within limits
- Custom constraint 

You can specify the planner via the `planning_pipeline` and `planner_id parameters`

### `move_group`
`move_group` is the main brain / coordinator node of MoveIt 2 (You send a goal → `move_group` figures out the motion → robot executes it)

You interact with `move_group` in 2 main ways:
1. **C++** (Using `move_group_interface` send motion commands in code)
2. **RViz GUI** (click goals in the MoveIt Motion Planning panel visualize plans)

**Configuration**: `move_group` reads robot data from ROS parameters: **URDF**
physical robot model (links, joints) stored in robot_description

**Robot Interface**: `move_group` listens on the `/joint_states` topic for determining the current state information 

---

### 2. Kinematics 

The default inverse kinematics plugin for MoveIt is configured using the KDL numerical Jacobian-based solver.

MoveIt 2 provides:
- **Forward Kinematics:** Computes end-effector pose from joint angles.
- **Inverse Kinematics:** Computes required joint angles from a target pose.

MoveIt 2 uses a plugin system, allowing users to write their own inverse kinematics algorithms

---

### 3. Collision Detection

Collision checking in MoveIt is configured inside a **Planning Scene** using the `CollisionWorld` object. It is mainly carried out using the FCL package (the primary collision checking library of MoveIt.)

MoveIt 2 continuously checks for:
- **Self-collisions:** Preventing the robot from hitting itself.
- **Environment collisions:** Avoiding walls, humans, or tables

MoveIt can detect collisions with:
- Meshes (.stl, .dae)
- Simple shapes: boxes, spheres, cylinders, cones, planes
- Octomap (3D map of the environment made from sensors)

---

### 4. 3D Perception
MoveIt 2 integrates perception systems to understand the robot’s environment.

Supported sensors:
- **RGB cameras**
- **Depth cameras**
- **LiDAR**

### RViz & Simulation

- **RViz2:** This is the visualization window where you see the "ghost" of the robot's planned path before the real robot moves.
- **Gazebo:** A physics simulator. It simulates gravity, friction, and weight so you can see if the robot will tip over or drop an object.

---

## Configure MoveIt 2 within the Dockerfile


#### Replace this to your "RUN apt-get install" section 

```
RUN apt-get update && apt-get install -y \
    bash \
    fluxbox \
    net-tools \
    novnc \
    supervisor \
    x11vnc \
    xvfb \
    xterm \
    python3-pip \
    python3-websockify \
    python3-rosdep \
    python3-colcon-common-extensions \
    python3-colcon-mixin \
    python3-vcstool \
    git \
    ros-humble-gazebo-ros-pkgs \
    ros-humble-ros2-controllers \
    ros-humble-xacro \
    ros-humble-robot-state-publisher \
    ros-humble-joint-state-publisher \
    ros-humble-ros2-control \
    && apt-get autoclean \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
```

#### Add these codes to your Dockerfile

```
# Initialize rosdep
RUN rosdep init || true && rosdep update

# Setup MoveIt 2 Workspace
WORKDIR /root/ws_moveit/src
RUN git clone -b humble https://github.com/moveit/moveit2_tutorials
RUN vcs import --recursive < moveit2_tutorials/moveit2_tutorials.repos

# Install Workspace Dependencies
WORKDIR /root/ws_moveit
RUN apt-get update && \
    rosdep install -r --from-paths src --ignore-src --rosdistro humble -y

# Build Workspace
RUN /bin/bash -c "source /opt/ros/humble/setup.bash && \
    export MAKEFLAGS='-j1 -l1' && \
    colcon build \
    --executor sequential \
    --parallel-workers 1 \
    --cmake-args \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_CXX_FLAGS='--param ggc-min-expand=20 --param ggc-min-heapsize=32768'"

# Final VNC & Entry Configuration
WORKDIR /root/
RUN ln -s /usr/share/novnc/vnc_lite.html /usr/share/novnc/index.html
RUN echo "source /root/ws_moveit/install/setup.bash" >> /root/.bashrc

```

#### Run the build
```
docker build --platform linux/amd64 -t moveit2-source-vnc .
```

---

###

### OMPL Planner

For a general-purpose robot arm in MoveIt 2, the most commonly recommended setup is **OMPL** (default)

#### Advantages:
- Finds paths quickly
- Works in many environments
- Flexible (Handles many robot types)
- Minimal tuning required

### CHOMP Planner
CHOMP optimization for complex obstacle environments (mathematically optimizes motion smoothness)

- Narrow spaces
- Dynamic obstacles
- Constrained motion
- Multi-step tasks

---

**Additional Information**: 
https://moveit.picknik.ai/main/doc/concepts/move_group.html
