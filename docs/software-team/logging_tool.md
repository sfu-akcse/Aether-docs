# Logging Tool

This document provides guidelines for a **unified logging system** in Project Aether, where **high-performance robot control (C/C++)** and **high-level AI logic (Python)** operate together. The goal is to efficiently monitor system behavior and debug issues across multiple languages.

---

## 1. Purpose

- **Unified Debugging:** Track and analyze logs from multi-language nodes in a single workflow.
- **Real-time Monitoring:** Observe system states such as robot arm coordinates, socket connections, and frame drops in real time.
- **Post-analysis:** Investigate failures using timestamps and structured log metadata.

---

## 2. Standard Log Levels

All modules must follow the **five standard log levels** below:

| Level | Name   | Description | Notes |
|-------|--------|-------------|-------|
| 1     | DEBUG  | Development-level details (e.g., variable values, coordinates) | Disabled in production |
| 2     | INFO   | Normal operations (e.g., "Connection established") | Tracks system flow |
| 3     | WARN   | Potential issues (e.g., delays, minor data loss) | No immediate failure |
| 4     | ERROR  | Functional failures (e.g., socket disconnect, IK (Inverse Kinematics) failure) | Affects execution |
| 5     | FATAL  | Critical failures (e.g., crash, emergency stop) | Immediate termination |

---

## 3. Implementation by Language

### Python (AI & Vision)

- Uses built-in `logging` module
- Outputs to both **console** and **log file** (`logs/combined.log`) 
- Includes timestamp, level, and module name

```python
import logging

logging.basicConfig(
    level=logging.INFO, # minimum level to display (DEBUG < INFO < ...)
    format='%(asctime)s [%(levelname)s] [%(name)s]: %(message)s',
    handlers=[
        logging.FileHandler("logs/combined.log"), # write logs to file
        logging.StreamHandler() # output logs to console
    ]
)

# Example
logger = logging.getLogger("VISION_NODE") # create logger for this module/node
logger.info("MediaPipe Hand Tracker started") # log a normal event
```

### C/C++ (Control & Socket)

- Uses lightweight rxi/log library
- Automatically includes file name, timestamp and line number
- Thread-safe for concurrent environments

```c++
#include "log.h"

// Example
void send_to_robot() {
    log_info("Socket Client: Sending target coordinates (X: 10.5)");
    if (send_failed) {
        log_error("%s:%d - Data transmission failed!", __FILE__, __LINE__);
    }
}
```

---

## 4. ROS 2 Integration

ROS 2 (Robot Operating System 2) is a middleware framework that enables communication between different components (nodes) in a robotic system.

Each module (e.g., AI, control, vision) runs as a **node**, and data is exchanged through ROS 2 communication mechanisms (topics, services).

This allows Python and C++ nodes to interact seamlessly without implementing custom networking logic.

When running on a **ROS 2 network**, logs from all nodes can be unified using ROS 2’s built-in logging system.

- **Python Logging in ROS 2:**

```python
node.get_logger().info("Message")
```

- **C++ Logging in ROS 2:**

```c++
RCLCPP_INFO(node->get_logger(), "Message");
```

Although Python and C++ use different logging APIs, ROS 2 automatically aggregates logs from all nodes into a single stream.

This enables centralized monitoring and debugging across the entire system regardless of implementation language.

#### When to Use ROS 2 Logging vs Native Logging

Logging approach should be chosen based on where the code is running:

- **ROS 2 Nodes**  
  Use ROS 2 logging APIs (`node.get_logger()`, `RCLCPP_INFO`) to enable centralized log aggregation and integration with ROS 2 tools (e.g., rqt_console).

- **Standalone / Utility Code**  
  Use native logging libraries (`logging` in Python, `log.h` in C/C++) for simplicity and independence from ROS 2.

This separation ensures both **system-wide observability** (via ROS 2) and **modular, reusable components** (via native logging).

---

## 5. Log Monitoring Tool

- **rqt_console** is a GUI tool for viewing and filtering logs from all ROS 2 nodes in one place.

It is especially useful when multiple nodes (Python and C++) are running simultaneously.

#### Key Features

- View logs from all nodes in a single interface  
- Filter logs by severity (DEBUG, INFO, WARN, ERROR, FATAL)  
- Filter logs by node name (e.g., VISION_NODE, CONTROL_NODE)  

#### Benefits

- Centralized visibility across Python and C++ nodes
- Filtering by log level (DEBUG → FATAL)
- Consistent timestamps and structured metadata

#### Usage

```bash
rqt_console
```

Using rqt_console improves debugging efficiency by providing a structured and searchable view of system logs.