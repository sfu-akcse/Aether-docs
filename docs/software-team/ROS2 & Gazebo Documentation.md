## 1. What is ROS2 and Gazebo and how is it structured?
### (1) ROS2
ROS is an open-source, meta-operating system for your robot. It provides the services you would expect from an operating system, including hardware abstraction, low-level device control, implementation of commonly-used functionality, message-passing between processes, and package management. It also provides tools and libraries for obtaining, building, writing, and running code across multiple computers.

The structures of ROS is 
- Nodes (Basic Execution Unit): The basic components of a ROS system. Each node is an independent process (program) and performs a specific functions like camera processing path planning or motor control etc. 

- ROS Graph (Runtime Structure): The network structure of nodes during execution. Nodes are connected in a peer-to-peer in loosely coupled manner. Nodes are not limited to running on a single computer, they can be distributed and run across multiple computers.

### (2) Gazebo
Gazebo is an open-source software that simulates robot motion, sensors, and physical interactions in realistic 2D/3D environments. It simulates how a robot would behave in real-world environments, tests what sensors (e.g., cameras, LiDAR) would perceive and uses physics engines to model gravity, collisions, and forces realistically.

Key Features of Gazebo is 
- Physics Simulation: Uses the ODE engine by default to compute gravity, collisions, and dynamics
- Graphics Rendering: Uses OpenGL and OGRE for realistic lighting, shadows, and textures
- Sensor Simulation: Supports sensors such as cameras, LiDAR, and Kinect-like devices
- Multiple Physics Engines: Supports not only ODE but also other engines like Bullet


## 2. How to set up ROS2 & Gazebo with docker?

### (1) How is it working?
It works with the help of supervisor and noVNC.
- supervisor: Docker’s original philosophy is “Run only one main process per container.” However, in our ROS 2 + GUI setup, we needed to run multiple processes inside a single container at the same time. So we “hired” a supervisor as our site manager. When the Docker container starts, the supervisor runs as the main process. It reads our task instructions (`supervisord.conf`) and runs multiple processes simultaneously inside the container.

- noVNC: A Docker container normally lives in a world with just a black terminal—there’s no concept of a physical monitor (display). So we had to artificially create one that captures that virtual screen in real time and streams it to our browser (e.g., `localhost:8080`) When we access it through Chrome on Windows or macOS, we can see the GUI.

### (2) How to make a docker image
1. Open the folder on the terminal
2. Enter `docker-compose up -d --build` on the terminal
3. Open localhost on your maching `http://localhost:8080` and press "connect"
4. Enter `source /opt/ros/humble/setup.bash` and `gazebo` on the new terminal
### (3) Resources
https://wiki.ros.org/docker/Tutorials/GUI

https://docs.ros.org/en/humble/How-To-Guides/Setup-ROS-2-with-VSCode-and-Docker-Container.html

https://discuss.google.dev/t/issues-deploying-gazebo-novnc-docker-on-cloud-run/192366

### (3) Codes
- docker-compose.yml
```yml
services:
  ros2-novnc:
    build: .
    container_name: ros2-sandbox
    ports:
      - "8080:8080"
    environment:
      # Adjust display resolution
      - DISPLAY_WIDTH=1280
      - DISPLAY_HEIGHT=800
    # volumes:
    #   - ./workspace:/root/workspace # when you want to share your code with local container
    restart: unless-stopped
```
- Dockerfile
```dockerfile
FROM osrf/ros:humble-desktop-full

ENV DEBIAN_FRONTEND=noninteractive
ENV DISPLAY=:0.0
ENV DISPLAY_WIDTH=1280
ENV DISPLAY_HEIGHT=800

# Install essential package
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
    ros-humble-gazebo-ros-pkgs \
    && apt-get autoclean \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# noVNC setting
WORKDIR /root/
RUN ln -s /usr/share/novnc/vnc_lite.html /usr/share/novnc/index.html

# Copying supervisor setting file
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Expose the port
EXPOSE 8080

# Starting supervisor when the container starts
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```
- supervisord.conf
```conf
[supervisord]
nodaemon=true

[program:xvfb]
command=Xvfb :0 -screen 0 "%(ENV_DISPLAY_WIDTH)s"x"%(ENV_DISPLAY_HEIGHT)s"x24 -listen tcp -ac
autorestart=true

[program:x11vnc]
command=x11vnc -forever -shared -display :0
autorestart=true

[program:fluxbox]
command=/usr/bin/startfluxbox
autorestart=true

[program:novnc]
command=websockify --web=/usr/share/novnc/ 8080 localhost:5900
autorestart=true

[program:xterm]
command=xterm -display :0
autorestart=true
```

## 3. What is the core logic?
### (1) ROS2
ROS2 is not a single engine, but a distributed communication system where nodes are connected and exchange data through standardized communication mechanisms.
- topic: It is the most fundamental communication method in ROS2. It follows a publisher–subscriber structure. When one node publishes data to a topic, other nodes that subscribe to that topic receive and process the data. This method is asynchronous, allowing data to flow simultaneously between multiple nodes. 

- service: It follows a request–response model and is synchronous. One node sends a request, another node processes it, and then returns a result. This is typically used for tasks that require an immediate response.

- action: It is similar to a service but is designed for long-running tasks. It includes a goal, intermediate feedback, and a final result. Actions can be canceled during execution and provide continuous feedback on progress.

Each node manages its configuration through "parameters."

Within this structure, data flows in patterns such as "node → topic → node", "node → service request → node → response" and this continuous flow of data enables the entire system to operate as a single robot. ("action" operates in a more complex way.)

### (2) Gazebo
Gazebo Server (gzserver) handles the core simulation logic. So core logic consists of everything that happens inside gzserver.

gzserver repeatedly executes the following process:
1. Scene Loading: Reads the world file (.world, SDF), loads robots, environment, lighting, and physics settings
2. Object & Sensor Initialization: Creates models, attaches sensors (e.g., cameras, LiDAR)
3. Physics Engine Execution: Computes gravity, collisions, and forces and updates the position and velocity of robots
4. Sensor Data Generation: Produces camera images, generates LiDAR point clouds, it could include configurable noise
5. Plugin Execution: Runs user-defined logic and enables robot control and integration with external systems
6. Repetition (Simulation Loop)

### (3) How does ROS2 and Gazebo works together?
ROS 2 and Gazebo work together as separate systems with different roles. Gazebo is responsible for creating a virtual environment and simulating robot motion and sensor data based on physical laws, while ROS 2 is responsible for making decisions about the robot’s behavior using that data.

These two systems are not directly connected but interact through ros_gz_bridge and the ROS 2 Simulation Interfaces. 
- ros_gz_bridge: It enables data exchange between the two systems, allowing sensor data generated in Gazebo to be sent to ROS 2, and control commands generated in ROS 2 to be sent back to Gazebo.
- Simulation Interfaces: It allow ROS 2 to directly control the simulation itself, such as starting or stopping the simulation, spawning robots, or querying their states.

Each direction of arrows of 2 systems represents 
- Gazebo → ROS2: Flow of sensor data from the simulated robot to ROS 2 for processing and decision-making.
- ROS2 → Gazebo: Flow of control commands from ROS 2 to the robot in the simulation, determining its actions and behavior.

## 4. Resources
#### ROS2
https://docs.ros.org/en/foxy/index.html

#### Gazebo
https://en.wikipedia.org/wiki/Gazebo_(simulator)
https://medium.com/@SuriNaren/gazebo-9448b264aef8
https://gazebosim.org/docs/latest/ros2_integration/

