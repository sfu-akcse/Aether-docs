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

For a general Python implementation, a practical starting point is newline-delimited JSON. That means:

- Every message is one JSON object
- Every message ends with `\n`
- The receiver reads until it finds a newline, then parses one complete message

For example:

```json
{"type":"command","x":120.0,"y":35.5,"z":210.0,"gripper":"open"}
```

This format is easy to inspect in logs and easy to parse in Python.

## How To Implement It

At a general level, implementation can be split into four steps.

### 1. Define the protocol

Before writing code, decide:

- Which side is the server
- Which IP address and port will be used
- What each message looks like
- Whether the server sends a reply
- How errors should be reported

Without this agreement, socket bugs become difficult to debug.

For the examples below, we will use this protocol:

- One program runs as the socket server
- Another program runs as the socket client
- Transport protocol: TCP
- Message format: newline-delimited JSON
- Port: `5000`

Each request sent from the client looks like this:

```json
{
  "type": "command",
  "x": 120.0,
  "y": 35.5,
  "z": 210.0,
  "gripper": "open"
}
```

The server replies with:

```json
{
  "status": "ok",
  "message": "Command received"
}
```

If the request is invalid, the server replies with:

```json
{
  "status": "error",
  "message": "Invalid command"
}
```

### 2. Create the socket connection

The client should:

- Create a TCP socket
- Connect to the server
- Confirm that the connection succeeds before sending messages

The server should:

- Bind to a port
- Listen for incoming connections
- Accept a client connection

### 3. Send messages in a consistent format

Once your application produces data to send, package it into the chosen message format and send it over the socket.

At this stage, keep the format small and readable. Human-readable messages are slower than compact binary messages, but they are much easier to test during development.

### 4. Handle failures safely

The system should be ready for:

- Connection refused
- Server not responding
- Broken connection
- Invalid message values
- Partial or corrupted messages

If the socket fails, the program should stop the current operation safely and fall back to whatever error-handling behavior makes sense for the application.

### Practical implementation guidance

#### Message boundaries

Across the Python and Real Python references, the same practical lessons appear again and again. The key idea is that `TCP` gives you a reliable byte stream, not automatic message boundaries. That means your program must decide where one message ends and the next begins. In Python, a very practical beginner-friendly choice is newline-delimited `JSON`:

- serialize one command as one JSON object
- append `\n`
- keep reading from `recv()` until a full line arrives
- parse only after the full line is complete

This is one of the most important implementation details because many first socket programs fail by assuming one `send()` always matches one `recv()`. In reality, `TCP` may split or combine data differently, so your code must reconstruct complete messages from the byte stream.

#### Start simple first

Another strong recommendation from the references is to start with `blocking sockets` and a simple `request-response` loop before trying more advanced designs. A first version should usually do this:

1. The server binds to a known host and port.
2. The server listens and accepts a connection.
3. The client connects.
4. The client sends one well-formed command.
5. The server validates the command and sends one reply.
6. Both sides log what was sent and received.

This keeps the first implementation easy to debug. If the basic connection, message framing, and validation are not working yet, moving to `async` or `multi-client` code usually makes debugging harder instead of easier.

#### Core socket operations

The references also emphasize the importance of choosing the right socket operations. In practice, a simple Python `TCP` implementation usually relies on:

- `socket.socket()` to create the socket
- `bind()`, `listen()`, and `accept()` on the server side
- `connect()` or `socket.create_connection()` on the client side
- `sendall()` instead of `send()` when you want to make sure the full buffer is transmitted
- `recv()` in a loop because one call may return only part of the data

For example, a minimal server setup in Python often starts like this:

```python
import socket


server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(("0.0.0.0", 5000))
server_socket.listen(1)

conn, addr = server_socket.accept()
print("Connected by:", addr)
```

In this example:

- `socket.socket(socket.AF_INET, socket.SOCK_STREAM)` creates an `IPv4 TCP` socket
- `bind()` attaches the socket to an IP address and port
- `listen()` tells the socket to wait for incoming connections
- `accept()` blocks until a client connects and then returns a new connection socket

On the client side, the connection step is usually much smaller:

```python
import socket


client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket.connect(("127.0.0.1", 5000))
```

You can also use `socket.create_connection()` as a convenient shortcut:

```python
import socket


client_socket = socket.create_connection(("127.0.0.1", 5000), timeout=3)
```

When sending a message, `sendall()` is generally safer than `send()` for simple application code:

```python
message = b'{"type":"command","x":120.0}\n'
client_socket.sendall(message)
```

When receiving data, the important point is to keep reading until a full message arrives:

```python
buffer = ""

while "\n" not in buffer:
    data = client_socket.recv(1024)
    if not data:
        raise ConnectionError("Connection closed")
    buffer += data.decode("utf-8")

line, buffer = buffer.split("\n", 1)
print("Received one full message:", line)
```

This pattern is important because `recv(1024)` does not mean "give me exactly one message." It means "give me up to 1024 bytes that are currently available."

#### Timeouts and failure handling

`Timeouts` are another important theme. A socket left in pure `blocking mode` can wait forever if the peer disappears or stops responding. For that reason, it is often safer to add `settimeout()` early, especially in robotics, where a stalled connection should not leave the system waiting indefinitely. Once timeouts are enabled, the program can detect failures earlier and switch to a safe fallback behavior.

For example:

```python
import socket


client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket.settimeout(3.0)

try:
    client_socket.connect(("127.0.0.1", 5000))
    data = client_socket.recv(1024)
except socket.timeout:
    print("Socket operation timed out")
```

In this example, `settimeout(3.0)` means operations such as `connect()` or `recv()` will raise a `socket.timeout` exception if they take longer than three seconds.

#### Separate transport from robot logic

The references also push a clear separation between `transport logic` and `application logic`. In other words:

- the socket layer should focus on connecting, sending, receiving, and parsing
- the application layer should decide what `x`, `y`, `z`, or `gripper` actually mean
- validation should happen before the command reaches the robot logic

That separation makes the code easier to test. You can first verify that the socket layer correctly receives a valid JSON line and returns a response. Only after that should you attach it to inverse kinematics, motion planning, or gripper control.

#### What to use as the project grows

As systems grow, the references point to three common next steps:

- `socketserver` when you want a more structured server with handler classes
- `selectors` when one process must watch multiple sockets efficiently
- `asyncio` streams when the program must handle network `I/O` together with other asynchronous tasks

These tools are valuable, but they are usually second-step tools. For most first implementations in a robotics project, a plain TCP client and server with clear framing, validation, logging, and timeouts is the best place to start.

### Suggested reading path

If you are implementing this for the first time, a practical order is:

1. Learn the request-response model and message framing idea.
2. Build a simple blocking TCP client and server.
3. Add newline-delimited JSON parsing.
4. Add validation and error replies.
5. Add timeouts and reconnection strategy.
6. Only then consider `socketserver`, `selectors`, or `asyncio`.

### References for deeper detail

- Python `socket` documentation: https://docs.python.org/3/library/socket.html
- Python Socket Programming HOWTO: https://docs.python.org/3/howto/sockets.html
- Python `socketserver` documentation: https://docs.python.org/3/library/socketserver.html
- Python `selectors` documentation: https://docs.python.org/3/library/selectors.html
- Python `asyncio` streams documentation: https://docs.python.org/3/library/asyncio-stream.html
- Real Python socket tutorial: https://realpython.com/python-sockets/
- Real Python `socket` reference: https://realpython.com/ref/stdlib/socket/

## Testing Tips

Testing in small steps is much better than trying the full pipeline all at once.

Recommended order:

1. Test a local socket connection on one machine.
2. Test sending a fixed sample request.
3. Confirm the server receives the exact message.
4. Connect the message to real application data.
5. Add any extra fields your protocol needs.
6. Add reconnection and error handling.

It is also helpful to print every sent and received message during early development.

It is also a good idea to test failure cases on purpose:

- Send invalid JSON
- Send a message with a missing field
- Disconnect the client while the server is waiting
- Stop the server while the client is still running

## Common Mistakes

Some common issues are:

- Client and server roles are reversed
- IP address or port number is wrong
- Message format is not clearly defined
- Numbers are sent in different units on each side
- One side expects newline-delimited messages but the other side does not send `\n`
- JSON is sent correctly, but the receiver tries to parse before a full message arrives
- Messages are sent too quickly for the server to process
- No safe behavior exists when the connection drops

## Suggested Team Checklist

Before integrating socket communication into the full robot system, the team should confirm:

- The server IP address and port are known
- The client/server roles are documented
- The message format is written down clearly
- Units or field meanings are agreed on
- The server has a safe default action on bad input

## Summary

Socket communication is a reusable pattern for connecting two programs over a network. The exact application code will change from project to project, but the development pattern stays the same: define the protocol clearly, implement both sides consistently, and handle failures safely. With the Python server and client examples above, another developer should be able to build, test, and extend a socket layer directly from this document.
