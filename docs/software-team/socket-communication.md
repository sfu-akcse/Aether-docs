---
title: Socket Communication
sidebar_position: 2
---

# Socket Communication

This page explains what a socket is, why we use socket communication in Project Aether, and how we can start implementing it for a physical robot arm.

## What Is a Socket?

A socket is one endpoint of a network connection between two programs. In practice, it lets one device send data to another device using an IP address and a port number.

For this project, a socket can be used to send robot arm commands from the main computer to the robot arm controller.

The most common model is:

- One side opens a port and waits for a connection.
- The other side connects to that port.
- After the connection is established, both sides can send and receive data.

In many robotics projects, TCP sockets are used because they are easier to reason about and more reliable than UDP for command delivery.

## Why We Use It in Aether

We starts from hand tracking on a laptop and then converts that information into robot arm motion.

At a high level, the flow is:

```text
Webcam / Hand Tracking
        |
        v
Computer Vision Node
        |
        v
ROS 2 / Inverse Kinematics
        |
        v
Socket Client  ----TCP---->  Robot Arm Controller
        |
        v
Send XYZ target / grab command
```

Simulation can happen inside Gazebo, but a physical robot arm needs a communication channel to receive commands from software running on the computer. A socket provides that channel.

## Basic Architecture

Socket communication usually involves two roles:

- A server, which waits for incoming connections
- A client, which connects to the server

The server normally listens on a specific IP address and port. The client uses that address and port to connect.

After the connection is established:

- The client can send requests, commands, or data
- The server can send responses, acknowledgements, or results
- In some systems, both sides can send data at any time

At a general level, the client is often responsible for:

- Starting the connection
- Sending a properly formatted message
- Waiting for a response when needed
- Handling connection failures

The server is often responsible for:

- Accepting connections
- Reading incoming messages
- Parsing and validating the data
- Returning a result or status message

This same pattern appears in many systems, including chat apps, games, web backends, IoT devices, and robotics projects.

## Communication Flow

Although implementations vary, socket communication often follows this general flow:

1. The server starts and listens on a port.
2. The client creates a socket and tries to connect.
3. Once connected, the client sends a message.
4. The server reads the message and processes it.
5. The server may send back a reply.
6. The client reads the reply if one is expected.
7. This exchange repeats until one side closes the connection.

In some cases, the connection stays open for a long time and many messages are exchanged. In other cases, the client connects, sends one message, receives one response, and disconnects immediately.

A simple TCP flow can be visualized like this:

```text
Client                                              Server
  |                                                   |
  |---------------------- SYN ----------------------->|
  |<------------------- SYN-ACK ----------------------|
  |---------------------- ACK ----------------------->|
  |                                                   |
  |              TCP connection established           |
  |                                                   |
  |-------------------- send data ------------------->|
  |<------------------- send reply -------------------|
  |-------------------- send data ------------------->|
  |<------------------- send reply -------------------|
  |                                                   |
  |---------------------- FIN ----------------------->|
  |<---------------------- ACK -----------------------|
  |<---------------------- FIN -----------------------|
  |---------------------- ACK ----------------------->|
  |                                                   |

```

This diagram shows the general idea of a TCP session:

- The client and server complete the TCP handshake
- Data can then move back and forth
- The connection is closed through a TCP termination sequence when communication is finished

## What Data Should Be Sent?

A socket only sends bytes, so both sides need to agree on how those bytes should be interpreted. This agreement is often called a protocol or message format.

The actual data depends on the application, but common examples include:

- Text messages
- Commands
- Sensor values
- Coordinates
- Status codes
- Timestamps
- JSON objects

It is usually best to start with a format that is easy to read and debug.

One simple JSON-style message could look like this:

```json
{
  "type": "command",
  "action": "move",
  "value": 42
}
```

Another plain-text message could look like this:

```text
COMMAND MOVE 42
```

No matter which format is used, both sides should agree on:

- What each field means
- What data types are expected
- Whether messages end with a newline or another delimiter
- How errors are reported

Clear message definitions are one of the most important parts of successful socket communication.

## How To Implement It

At a general level, implementation can be split into four steps.

### 1. Define the protocol

Before writing code, decide:

- Which side is the server
- Which IP address and port will be used
- What each message looks like
- Whether the controller sends a reply
- How errors should be reported

Without this agreement, socket bugs become difficult to debug.

### 2. Create the socket connection

The client should:

- Create a TCP socket
- Connect to the controller
- Confirm that the connection succeeds before sending commands

The server should:

- Bind to a port
- Listen for incoming connections
- Accept a client connection

### 3. Send commands in a consistent format

Once inverse kinematics produces a valid target, package the output into the chosen message format and send it over the socket.

At this stage, keep the format small and readable. Human-readable messages are slower than compact binary messages, but they are much easier to test during development.

### 4. Handle failures safely

The system should be ready for:

- Connection refused
- Controller not responding
- Broken connection
- Invalid command values
- Partial or corrupted messages

If the socket fails, the program should avoid sending uncontrolled robot motions and should fall back to a safe state.

## Example Pseudocode

This example is intentionally general and does not depend on a specific programming language.

```text
create TCP socket
connect to robot controller

while system is running:
    get target position from IK module
    format message
    send message
    optionally receive status

close socket
```

## Testing Tips

Testing in small steps is much better than trying the full pipeline all at once.

Recommended order:

1. Test a local socket connection on one machine.
2. Test sending a fixed sample command.
3. Confirm the controller receives the exact message.
4. Connect the message to the inverse kinematics output.
5. Add the gripper command.
6. Add reconnection and error handling.

It is also helpful to print every sent and received message during early development.

## Common Mistakes

Some common issues are:

- Client and server roles are reversed
- IP address or port number is wrong
- Message format is not clearly defined
- Numbers are sent in different units on each side
- Messages are sent too quickly for the controller to process
- No safe behavior exists when the connection drops

## Suggested Team Checklist

Before integrating socket communication into the full robot system, the team should confirm:

- The controller IP address and port are known
- The client/server roles are documented
- The message format is written down clearly
- Units are agreed on, such as millimeters or centimeters
- The controller has a safe default action on bad input

## Summary

In Project Aether, socket communication is the bridge between software running on the laptop and the physical robot arm. The exact code may change depending on the robot controller, but the core idea stays the same: establish a connection, send well-defined commands, and handle failures safely.
