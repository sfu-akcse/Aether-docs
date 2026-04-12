---
title: Power and Signal Flow
sidebar_label: 1. Power & Signal Flow
---
## base design and power intergration 


## 1.Stepper vs. Servo
The selection of an actuator depends on the trade-off between open-loop simplicity and closed-loop precision.

### **Stepper Motor (Open Loop Control)**
* **Architecture:** Formed with two rotor sections (gears) connected with a central shaft and a stator. These two gears are **mismatched (mechanically offset)** with each other, and a permanent magnet is sandwiched between them.
* **Mechanism:** When the stator is magnetized by a pulse from the controller, the magnetic flux forces the rotor to move by a discrete "step."
* **The Feedback Gap:** There is **no feedback** in a standard stepper motor. If the output encounters an error or misses a step (stall), the system will not adjust, leading to positioning drift.
* **Torque Profile:** * **High Torque at Low Speed:** It is Ideal for holding positions.
    * **Low Torque at High Speed:** Due to **Back-EMF** and inductance, torque drops sharply as RPM increases.
    

### **Servo Motor (Closed Loop Control)**
* **Architecture:** Unlike the stepper motor, the magnet itself becomes the rotator. The critical component is the **integrated encoder**.
* **Mechanism:** The encoder provides constant real-time feedback, making the servo a **Closed Loop** system.
* **Self-Correction:** The driver remains active and adjusts the power/current until the output matches the input. If an external force moves the shaft, the servo automatically corrects it.
* **Torque Profile:** Maintains nearly **constant torque** from low speed to high speed (Flat Torque Curve).
    
** we chose to use servo motor 


## 2. Communication & Power Integration
### **UART Driver & Protocol**
Unlike PWM, which relies on duty-cycle timing, the **UART (Universal Asynchronous Receiver-Transmitter)** focus is on transferring complex data packets.
* **Bi-directional:** Uses **TX (Transmit)** and **RX (Receive)** lines to give commands and take digital signals (telemetry) back from the motor.
* **Data Feedback:** Allows the controller to monitor motor temperature, current position, and load status in real-time.

### Driver: Bus Servo Driver HAT (A)** https://www.waveshare.com/wiki/Bus_Servo_Driver_HAT_(A)?srsltid=AfmBOoqzLAecP4-8MwVUmfs0UlN7n1-4paxMVtnfELZcp2r9a28Z9GVW 

This specific driver acts as the bridge between the Raspberry Pi's logic and the high-current requirements of the bus servos.
* **Daisy Chain Topology:** By using one driver and one power supply, you can connect all motors in a series "chain."
* **Efficiency:** Each motor is assigned a unique ID. You can control multiple degrees of freedom (DoF) using only a single data wire sequence, significantly reducing wiring complexity.

---

## 3. Structural Design: Case v1 (Sleeve & Case)
a "Sleeve & Case" design, allowing the internal core to be easily removed for maintenance. The internal stack is optimized for thermal management and stability.

### **Level-Based Integration**

| **Level 3 (Top)** | **UART Hat Driver** | **Connectivity:** Clean routing for daisy-chained motor wires. |
| **Level 2 (Mid)** | **Raspberry Pi & Active Cooler** | **Thermals:** High-performance processing requires active airflow to prevent throttling. |
| **Level 1 (Base)** | **Battery Pack** | **Stability:** The heaviest object is at the bottom to lower the Center of Gravity ($CoG$). |

**I was thinking to put the arm over the case(on the level3) at first. But we can discuss about this. if we don't want this**
-> ## seperate base 
putting raspbeery pi case + power supply + base motor

### **Stability Calculation**
To prevent tipping during high-acceleration motor movements, we have to calculate the torque on the case after we choose the size and length of the arm.

---

## 4. Power & Signal Flow

**it requires lots of watt to perform both raspberry pi and multiple servo motors. at first I thought that we can just use one battery pack but we decided to use seperate power supply**

1. battery pack for raspberry pi https://www.amazon.ca/dp/B0BQC2WNR8?ref=nb_sb_ss_w_as-reorder_k0_1_4&amp=&crid=1VEIYTFUHZABN&amp=&sprefix=ups+

2. power supply : https://www.amazon.ca/VAYALT-Switching-Converter-Universal-Regulated/dp/B0DXL18XNW/ref=asc_df_B0DXL18XNW?mcid=81253a0919fb3a7e9af3834406f69ef1&tag=googleshopc0c-20&linkCode=df0&hvadid=706827341411&hvpos=&hvnetw=g&hvrand=14969579037107097618&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9001527&hvtargid=pla-2422946907086&hvocijid=14969579037107097618-B0DXL18XNW-&hvexpln=0&gad_source=1&th=1

w