# Sockets Documentation 

This document describes the communication architecture for the Aether Robot Arm, explains the rationale behind each technology decision, and traces the data path from the computer vision pipeline through to the physical servo motors.

---

## Subsystems Overview

The following subsystems must work together in real time:

- Hand tracking via MediaPipe
- XY coordinate detection
- Z coordinate (depth) detection
- Wrist rotation detection
- Grab / pinch detection
- Inverse kinematics solver
- Motor and PWM control
- Raspberry Pi 5 hardware interface

---

## System Architecture

The system is divided into two physical compute nodes that communicate wirelessly over a local network.

### Main PC / Laptop

Runs all perception and planning workloads:

- MediaPipe hand landmark detection
- XY, Z, wrist rotation, and grab-state calculations
- Inverse kinematics solver
- ROS2 topic publishers

### Raspberry Pi 5

Handles all real-time hardware interaction:

- ROS2 topic subscribers
- Motor command generation
- PWM and GPIO output
- Servo driver communication

### High-Level Data Flow

```
Camera
   │
MediaPipe (landmark detection)
   │
   ├─ XY Coordinate Node
   ├─ Z Coordinate Node
   ├─ Wrist Rotation Node
   └─ Grab Detection Node
   │
ROS2 Topics  (DDS / TCP networking)
   │
WiFi / Ethernet
   │
Raspberry Pi 5  ─  ROS2 Subscribers
   │
Motor Control Node
   │
PWM / Servo Driver
   │
Robot Arm
```

---

## Communication Protocols

### TCP (Transmission Control Protocol)

TCP is a reliable, connection-oriented protocol that ensures all transmitted data arrives correctly and in order.

For the Aether robot arm, TCP is critical because the system continuously sends:

- Hand coordinates
- Wrist rotation data
- Grab state information
- Motor control commands

If packets are lost or arrive out of order, the robotic arm could move unpredictably, jitter, or receive incorrect motor positions.

TCP automatically handles:

- Packet ordering
- Error checking
- Retransmission of lost packets

**Example TCP server:**

```python
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("0.0.0.0", 5000))
server.listen()
client, addr = server.accept()
data = client.recv(1024)
```

**Example TCP client:**

```python
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(("192.168.1.10", 5000))
client.send(b"x=120")
```

---

### UDP (User Datagram Protocol)

UDP is a faster, connectionless protocol that prioritizes speed over reliability.

Unlike TCP, UDP does not guarantee:

- Packet delivery
- Packet ordering
- Retransmission of lost data

UDP is commonly used for live video streaming, online games, and real-time sensor feeds where occasional packet loss is acceptable.

**UDP was not selected** for this project because robotic arm control requires stable, reliable communication. Missing or corrupted motor commands could cause incorrect arm movement and reduce system stability.

**Example UDP receiver:**

```python
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server.bind(("0.0.0.0", 5000))
data, addr = server.recvfrom(1024)
```

**Example UDP sender:**

```python
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
client.sendto(b"x=120", ("192.168.1.10", 5000))
```

---

### TCP vs UDP Comparison

| Characteristic | TCP | UDP |
|---|---|---|
| Connection Type | Connection-oriented | Connectionless |
| Reliability | Guaranteed delivery | No delivery guarantee |
| Packet Ordering | Packets arrive in order | Packets may arrive out of order |
| Error Handling | Automatic retransmission | No retransmission |
| Speed | Slightly slower | Faster with lower overhead |
| Best Use Cases | Robot control, coordinate data | Video streaming, gaming, sensor feeds |
| Suitability for Aether |  Highly suitable | Not recommended |

---

## ROS2 Middleware

### What is ROS2?

ROS2 (Robot Operating System 2) is an open-source robotics middleware framework that provides a standardised communication layer for distributed robotic systems. Unlike raw sockets, ROS2 abstracts away low-level networking details and organises the system into discrete, independently developed components.

Internally, ROS2 uses **DDS (Data Distribution Service)** as its transport layer, which supports both TCP-like reliable delivery and UDP-like best-effort modes.

### Nodes

Every independent robotic component is implemented as a ROS2 node — a self-contained process with a single, well-defined responsibility.

Nodes in the Aether project:

- XY Coordinate Detection Node
- Z Coordinate (Depth) Detection Node
- Wrist Rotation Node
- Grab Detection Node
- Inverse Kinematics Node
- Motor Controller Node

### Topics, Publishers, and Subscribers

Nodes communicate through named channels called **topics** using a publisher-subscriber pattern. A publisher writes data to a topic; any number of subscribers can read from that same topic without the publisher needing to know about them.

This decoupling means new tools (logging, visualisation, simulation) can be connected by adding subscribers — with zero changes to existing nodes.

**Example flow:**

```python
# PC publishes
publisher.publish(xy_msg)   # → /xy_coordinates

# Raspberry Pi subscribes
create_subscription(CoordinateMsg, '/xy_coordinates', callback)
```

### Why ROS2 Over Raw Sockets

Raw socket implementations require manual coding of packet framing, error recovery, reconnection logic, multithreading, and message serialisation. In a project with six or more concurrent data streams this becomes difficult to maintain and scale.

ROS2 provides all of that infrastructure out of the box, plus:

- Automatic node discovery across the network
- Standardised message types and serialisation
- Built-in support for Gazebo simulation and MoveIt2 motion planning
- Easy integration of logging, monitoring, and visualisation (RViz)
- Scalability — new subscribers attach without touching publisher code

---

## Data Flow Through the System

### Step 1 — Camera & MediaPipe

The laptop camera captures frames continuously. MediaPipe processes each frame and returns a set of 21 hand landmarks representing finger-joint positions in normalised image coordinates.

```python
landmarks = mediapipe.detect(frame)
```

### Step 2 — Feature Extraction Nodes

Four dedicated ROS2 nodes extract specific features and publish them to separate topics:

| Node | Output | Topic |
|---|---|---|
| XY Coordinate Node | Horizontal and vertical hand position | `/xy_coordinates` |
| Z Coordinate Node | Depth estimation from landmark spread | `/z_coordinate` |
| Wrist Rotation Node | Wrist orientation angle | `/wrist_rotation` |
| Grab Detection Node | Open/closed hand state | `/grab_state` |

### Step 3 — ROS2 Transmission

When any node calls `publisher.publish(msg)`, ROS2 serialises the message, routes it through the DDS transport layer over the local network, and delivers it to every matching subscriber automatically. No manual socket management is required.

### Step 4 — Raspberry Pi Subscription

The Raspberry Pi runs subscriber nodes that receive the latest values as they are published:

```python
def callback(msg):
    x = msg.x
    y = msg.y
    # compute IK and send motor commands
```

### Step 5 — Inverse Kinematics & Motor Control

The motor control node on the Pi receives all four data streams and:

1. Solves inverse kinematics to convert Cartesian coordinates into joint angles
2. Generates PWM duty cycles for each servo
3. Sends signals to the servo driver board
4. Drives the physical robot arm to the computed pose

---

## Network Transport Methods

### ROS2 over WiFi / Ethernet

When a ROS2 node publishes a message, the data path is:

```
ROS2 Publisher
        ↓
DDS Middleware
        ↓
TCP/UDP Network Packets
        ↓
WiFi or Ethernet
        ↓
Raspberry Pi 5
        ↓
ROS2 Subscriber
```

ROS2 automatically handles node discovery, networking, message serialisation, packet routing, and delivery to subscribers.

### WiFi

**Architecture:**
```
Laptop ROS2 Node → WiFi Router → Raspberry Pi 5
```

The laptop and Raspberry Pi must be on the same network with ROS2 running and DDS discovery enabled.

| | |
|---|---|
| Advantages | Wireless operation, no tethered cable, easier robot movement |
| Disadvantages | Possible latency spikes, wireless interference, less stable than Ethernet |

### Ethernet

**Architecture:**
```
Laptop → Ethernet Cable → Raspberry Pi 5
```

ROS2 communication remains identical at the software level. Only the physical transport changes.

| | |
|---|---|
| Advantages | Lower latency, higher reliability, lower packet loss, preferred for robotics |
| Disadvantages | Requires physical cable, reduced mobility |

### USB Serial

USB transfers raw serial data directly between devices:

```python
# PC sends
ser.write(b'45,90,1\n')

# Microcontroller reads
Serial.readStringUntil('\n');
```

USB serial is simple and effective for Arduino projects and direct motor control, but does not provide distributed networking, automatic discovery, or scalable multi-node architectures. It was considered less suitable for Aether.

### UART

UART is a low-level serial protocol commonly used between microcontrollers and embedded systems, transferring data one bit at a time over TX/RX lines.

```
TX  →  RX
RX  ←  TX
GND ↔  GND
```

UART is useful for low-level hardware communication and debugging, but requires manual implementation of packet structures, parsing, synchronisation, and error handling.

### Raw Sockets (Evaluated, Not Selected)

Before selecting ROS2, raw TCP/UDP sockets were evaluated:

```
Laptop → TCP Socket → Raspberry Pi
```

```python
socket.send(json_data)
```

While functional, raw sockets become increasingly difficult to manage as the number of subsystems grows due to:

- Manual networking and reconnection code
- Packet framing complexity
- Multithreading management
- Scalability issues
- Increased debugging difficulty

Because the Aether project contains multiple independent subsystems and future expansion plans, **ROS2 was selected** over raw sockets.
