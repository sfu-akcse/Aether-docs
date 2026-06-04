# ROS2 Gazebo Simulation

A ROS2-based simulation environment for the AKCSE robotic arm, enabling software development, motion testing, and joint control validation before physical hardware deployment.

---

## Overview

The full pipeline runs from camera input through hand state estimation into joint control:

```
Camera → hand_tracking_node → hand_state_node → /hand_state → IK node → /arm_controller/joint_trajectory → Gazebo
```

The simulation side converts SolidWorks CAD geometry into a controllable Gazebo model:

```
SolidWorks → STL → URDF → Gazebo → ros2_control → Joint Control
```

---

## Prerequisites

- Docker and Docker Compose installed
- An X server (for GUI — Gazebo and RViz):
  - **Linux**: already available, just run `xhost +local:docker`
  - **macOS**: install [XQuartz](https://www.xquartz.org/)
  - **Windows**: install VcXsrv or use WSL2 with WSLg

---

## Getting Started

### Setup

```bash
git clone https://github.com/your-org/robot_arm_ws.git
cd robot_arm_ws

xhost +local:docker      # allow Docker to open GUI windows
docker compose build
```

### Run

```bash
docker compose up        # launches Gazebo, loads URDF, starts controllers
```

### Send Joint Commands

```bash
docker exec -it ros2_arm_sim bash

ros2 topic pub /arm_controller/joint_trajectory \
  trajectory_msgs/msg/JointTrajectory \
  "{joint_names: ['shoulder_joint', 'elbow_joint', 'wrist_joint', 'gripper_joint'],
    points: [{positions: [0.5, 1.2, -0.3, 0.0], time_from_start: {sec: 2}}]}"
# gripper_joint: 0.0 = closed, 0.04 = open
```

### Stop

```bash
docker compose down
```

---

## Configuration Files

### docker-compose.yml

```yaml
version: "3.8"

services:
  ros2_sim:
    build: .
    container_name: ros2_arm_sim
    environment:
      - DISPLAY=${DISPLAY}
      - ROS_DOMAIN_ID=0
    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix
      - ./src:/ros2_ws/src
    network_mode: host
    command: >
      bash -c "source /opt/ros/humble/setup.bash &&
               source /ros2_ws/install/setup.bash &&
               ros2 launch arm_sim simulation.launch.py"
```

### Dockerfile

```dockerfile
FROM osrf/ros:humble-desktop

RUN apt-get update && apt-get install -y \
    ros-humble-gazebo-ros-pkgs \
    ros-humble-ros2-control \
    ros-humble-ros2-controllers \
    ros-humble-joint-trajectory-controller \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /ros2_ws
COPY src/ src/

RUN bash -c "source /opt/ros/humble/setup.bash && colcon build --symlink-install"
RUN echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc && \
    echo "source /ros2_ws/install/setup.bash" >> ~/.bashrc
```

---

## Project Structure

```
robot_arm_ws/
├── docker-compose.yml
├── Dockerfile
└── src/
    ├── robot_interfaces/          # custom ROS2 messages
    │   └── msg/
    │       └── HandState.msg
    ├── hand_tracking/             # camera + hand state nodes
    │   └── hand_tracking/
    │       ├── camera_node.py
    │       ├── hand_tracking_node.py
    │       └── hand_state_node.py
    └── robot_description/         # simulation
        ├── meshes/
        │   ├── base.stl
        │   ├── upper_arm.stl
        │   ├── forearm.stl
        │   ├── wrist.stl
        │   └── gripper.stl
        ├── urdf/
        │   └── arm.urdf.xacro
        ├── config/
        │   └── controllers.yaml
        └── launch/
            └── simulation.launch.py
```

---

## Required Files

### STL Files (`meshes/`)

Exported directly from SolidWorks. Define the visual geometry of each robot link.

- `base.stl`
- `upper_arm.stl`
- `forearm.stl`
- `wrist.stl`
- `gripper.stl`

> STL files define appearance only — not joint motion, structure, or control logic.

---

### URDF / Xacro (`urdf/arm.urdf.xacro`)

Defines the robot's kinematic structure: links, joints, mass, inertia, and joint limits.

**Link example:**

```xml
<link name="upper_arm">
  <visual>
    <geometry>
      <mesh filename="package://robot_description/meshes/upper_arm.stl"/>
    </geometry>
  </visual>
</link>
```

**Joint example:**

```xml
<joint name="shoulder_joint" type="revolute">
  <parent link="base_link"/>
  <child link="upper_arm"/>
  <axis xyz="0 0 1"/>
  <limit lower="-1.57" upper="1.57" effort="10" velocity="1.0"/>
</joint>
```

**Gripper joint:**

```xml
<joint name="gripper_joint" type="prismatic">
  <parent link="wrist"/>
  <child link="gripper"/>
  <axis xyz="1 0 0"/>
  <limit lower="0.0" upper="0.04" effort="5" velocity="0.5"/>
  <!-- 0.0 = fully closed, 0.04 = fully open (metres) -->
</joint>
```

> Use `type="prismatic"` for a parallel jaw gripper (linear open/close). Use `type="revolute"` if your gripper fingers rotate.

---

### YAML Controller Config (`config/controllers.yaml`)

Configures `ros2_control` — maps joints to controllers and exposes ROS2 topics.

```yaml
controller_manager:
  ros__parameters:
    update_rate: 100
    arm_controller:
      type: joint_trajectory_controller/JointTrajectoryController

arm_controller:
  ros__parameters:
    joints:
      - shoulder_joint
      - elbow_joint
      - wrist_joint
      - gripper_joint
    command_interfaces:
      - position
    state_interfaces:
      - position
      - velocity
```

This generates the topic: `/arm_controller/joint_trajectory`

**How the topic name is generated:**

| Controller Name | Generated Topic |
|---|---|
| `arm_controller` | `/arm_controller/joint_trajectory` |
| `robot_controller` | `/robot_controller/joint_trajectory` |
| `my_arm` | `/my_arm/joint_trajectory` |

The `/joint_trajectory` suffix is fixed and hardcoded into `joint_trajectory_controller`. Only the prefix is configurable.

**How the subscription works:**

You never write a subscriber yourself. When `joint_trajectory_controller` loads, it automatically subscribes to the generated topic:

```
YAML loads joint_trajectory_controller
        ↓
joint_trajectory_controller auto-subscribes to /arm_controller/joint_trajectory
        ↓
your publisher just needs to publish to that topic
```

> The most common mistake is renaming the controller in the YAML but not updating the topic name in the publisher, or vice versa.

---

### Launch File (`launch/simulation.launch.py`)

```bash
ros2 launch arm_sim simulation.launch.py
```

Responsibilities:
- Starts Gazebo
- Spawns the robot from URDF
- Loads and activates controllers

---

## Hand State Pipeline

The hand tracking system runs as three internal modules but publishes a single combined ROS2 topic, keeping XYZ, wrist angle, and grab state on the same camera frame.

```
camera_node
     ↓
hand_tracking_node
     ↓
hand_state_node
  ├─ calculates XYZ
  ├─ calculates wrist angle
  └─ calculates grab state
          ↓
   publishes /hand_state
```

### Custom Message (`HandState.msg`)

```
float32 x
float32 y
float32 z
float32 wrist_angle
bool    grab
```

Place this in `robot_interfaces/msg/HandState.msg` and add the package as a dependency.

### Why One Topic

Publishing separate `/xyz`, `/wrist`, `/grab` topics forces the robot controller to synchronize across them — there's no guarantee they arrive in the same frame or in order. A single `/hand_state` message eliminates that problem: one timestamp, one subscriber, no sync logic needed.

### Publisher Example (`hand_state_node.py`)

```python
from robot_interfaces.msg import HandState

pub = self.create_publisher(HandState, "/hand_state", 10)

msg = HandState()
msg.x = 0.45
msg.y = 0.12
msg.z = 0.30
msg.wrist_angle = -0.3
msg.grab = True

pub.publish(msg)
```

### Subscriber Example (IK / robot control node)

```python
from robot_interfaces.msg import HandState

def hand_state_callback(self, msg):
    # All values are guaranteed from the same frame
    x, y, z = msg.x, msg.y, msg.z
    wrist    = msg.wrist_angle
    grabbing = msg.grab

self.create_subscription(HandState, "/hand_state", hand_state_callback, 10)
```

---

## Joint Control Flow

**Step 1** — IK node receives `/hand_state` and calculates joint angles:

```
x, y, z, wrist_angle → IK solver →

shoulder = 0.5 rad
elbow    = 1.2 rad
wrist    = -0.3 rad

grab = True  → gripper_joint = 0.0  (closed)
grab = False → gripper_joint = 0.04 (open)
```

**Step 2** — Python publisher sends a `JointTrajectory` message:

```python
from trajectory_msgs.msg import JointTrajectory, JointTrajectoryPoint

pub = self.create_publisher(JointTrajectory, "/arm_controller/joint_trajectory", 10)

GRIPPER_OPEN   = 0.04
GRIPPER_CLOSED = 0.0

def publish_command(shoulder, elbow, wrist, grab):
    msg = JointTrajectory()
    msg.joint_names = ["shoulder_joint", "elbow_joint", "wrist_joint", "gripper_joint"]

    point = JointTrajectoryPoint()
    point.positions = [
        shoulder,
        elbow,
        wrist,
        GRIPPER_CLOSED if grab else GRIPPER_OPEN
    ]
    point.time_from_start.sec = 2

    msg.points = [point]
    pub.publish(msg)
```

**Step 3** — `joint_trajectory_controller` receives the message automatically.

**Step 4** — Controller maps names to YAML config → URDF joints → Gazebo actuators.

**Step 5** — All four joints move, including the gripper.

---

## Joint Name Consistency

Joint names must be identical across all three files or Gazebo cannot map commands correctly.

| File | Usage |
|---|---|
| `arm.urdf.xacro` | `<joint name="shoulder_joint" .../>` ... `<joint name="gripper_joint" .../>` |
| `controllers.yaml` | `joints: [shoulder_joint, ..., gripper_joint]` |
| Python publisher | `msg.joint_names = ["shoulder_joint", ..., "gripper_joint"]` |

---

## Summary

| Goal | Components Needed |
|---|---|
| Display robot only | STL + URDF |
| Display and move robot | STL + URDF + YAML + Launch File |
| Full pipeline | All above + hand_tracking nodes + robot_interfaces msg + IK node |
