# Mechanical Team Overview

## Core Objective
To engineer a robotic system capable of **live coordinate tracking** and precision tactile manipulation, following human hand movements in real-time.

---

## Technical Milestones

### I. Physical Design & Kinematics
* **Mechanical Design:** Design a robotic arm capable of grabbing tasks.
* **Degrees of Freedom (DoF):** Define joint configurations to ensure fluid motion and operational stability during coordinate-based tracking.

### II. Firmware & Connectivity
* **Communication Bridge:** Configure **Raspberry Pi** as a socket client to receive high-frequency coordinate data from the control software.
* **Signal Processing:** Develop the firmware layer to translate incoming software coordinates into hardware-level execution.

### III. Actuation & Motion Control
* **Hardware Integration:** Interface Raspberry Pi with **Serial Servo/Stepper controllers** to manage precise positioning and power delivery.
* **Actuation Logic:** Implement synchronized multi-axis movement logic to translate live hand coordinates into mechanical displacement.

### IV. Functional Flow (Example)
1. **Detection:** Vision algorithm identifies a "fist" gesture or specific hand coordinates.
2. **Transmission:** Data packets are sent via socket protocol to the Raspberry Pi.
3. **Execution:** Local application software processes the coordinates and triggers the **grabbing servo** for physical actuation.

---

## Key Performance Indicators (KPIs)
* **Latency:** Minimal delay between hand movement and mechanical response.
* **Precision:** Repeatable accuracy in joint positioning.
* **Load Capacity:** Successful manipulation of target payloads without motor stall.