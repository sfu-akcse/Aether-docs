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

## 4. Power & Signal Flow

**it requires lots of watt to perform both raspberry pi and multiple servo motors. at first I thought that we can just use one battery pack but we decided to use seperate power supply**

1. battery pack for raspberry pi https://www.amazon.ca/dp/B0BQC2WNR8?ref=nb_sb_ss_w_as-reorder_k0_1_4&amp=&crid=1VEIYTFUHZABN&amp=&sprefix=ups+

2. power https://www.amazon.ca/DIGISHUO-Transformer-110V-240V-Switching-Converter/dp/B0986S4Y24/ref=asc_df_B0986S4Y24?mcid=1da16aff9b573012bc7d80ee5c039234&tag=googleshopc0c-20&linkCode=df0&hvadid=706725384612&hvpos=&hvnetw=g&hvrand=10273603512019279346&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9194213&hvtargid=pla-1622320292074&psc=1&hvocijid=10273603512019279346-B0986S4Y24-&hvexpln=0&gad_source=1

# Power System Integration: Motor Power Rationale

This document outlines the power requirements, calculations, and selection rationale for the robotic arm's actuator system.

## 1. System Configuration
* **Actuators:** 7 x Waveshare ST3215 Serial Bus Servos
* **Configuration:** 2 motors for the base support, 5 motors for the arm and gripper.
* **Operating Voltage:** 12.0V (Standard)

## 2. Power Consumption Calculations

We evaluate the system's power needs based on two primary scenarios: the absolute peak (Worst-Case) and the standard operational load.

### 2.1 Peak Power Consumption (Worst-Case Scenario)

The worst-case occurs during a "Locked-Rotor" (Stall) event where all motors encounter maximum resistance simultaneously.

* Voltage (V): 12V
* Stall Current (I_stall): 2.7A

Peak Power per Motor (P_peak):
P_peak = V x I_stall = 12V x 2.7A = 32.4W

Total System Peak Power (P_total_peak):
P_total_peak = 32.4W x 7 = 226.8W

### 2.2 Realistic Operational Load

We assume a standard task of manipulating a 1kg bottle at an arm length of 15cm. The Torque Constant (K_T) of 11kg.cm/A is used to derive the actual current draw.

* Torque Constant (K_T): 11kg.cm/A

Required Torque (tau):
tau = 1kg x 15cm = 15kg.cm

Current per Motor (I_load):
I_load = tau / KT = 15kg.cm / 11kg.cm/A = 1.36A

Operational Power per Motor (P_load):
P_load = 12V x 1.36A = 16.32W

Total Operational Power (P_total_load):
P_total_load = 16.32W x 7 = 114.24W

## 3. Power Supply Selection: 300W vs. 500W

Based on the calculations above, a 300W Power Supply is selected. It provides a healthy safety margin for standard tasks (approx. 38% utilization) while remaining capable of handling absolute peak loads (226.8W).

### Why we avoid over-specifying (e.g., 500W Power Bank)

1. **Cost Inefficiency** Engineering design requires a balance between performance and budget. A 500W unit is significantly more expensive than a 300W unit without providing any functional advantage for a 227W peak system.

2. **Spatial and Weight Constraints** High-spec power components are inherently larger and heavier to dissipate more heat and manage higher currents. For a base footprint of 10.0cm x 8.5cm, a 500W unit would cause significant physical interference and increase the overall weight, lowering the robot's mobility.

3. **Efficiency Loss at Low Loads** Switching Mode Power Supplies (SMPS) reach peak efficiency at roughly 50% to 80% of their rated capacity.
   * **300W PSU:** Operates near its efficiency "sweet spot" during normal tasks.
   * **500W PSU:** Would operate at very low loads (less than 25%), leading to significant energy loss as waste heat and reducing the overall efficiency of the power system.

   
   
   
# Raspberry Pi 5 Power Strategy: Comparison & Analysis

## 1. Strategy 1: Dedicated Battery System (Independent Power)
The RPi 5 is powered by its own separate battery pack (e.g., Li-ion pack or Power Bank).

### Advantages (Pros)
* **Total Electrical Isolation:** The RPi 5 is completely shielded from the electrical noise, voltage spikes, and Back EMF (electromagnetic interference) generated by the 7 high-torque servo motors.
* **Elimination of Brownouts:** Even if the motors draw a massive surge of current and cause a momentary voltage sag in the main PSU, the RPi 5 remains stable and will not reboot.
* **Simplified Debugging:** If the motor system fails or short-circuits, the brain (RPi 5) stays alive, allowing you to check logs and diagnose the issue without losing the connection.

### Disadvantages (Cons)
* **Maintenance Complexity:** You must manage and charge two separate power sources.
* **Weight and Space:** Adding a second battery increases the total weight and occupies more volume within the base.



## 2. Strategy 2: Integrated PSU System (Shared Power via Buck Converter)
The RPi 5 shares the 12V 300W PSU with the motors, using a DC-DC Step-down (Buck) converter.

### Advantages (Pros)
* **Single Power Source Management:** You only need to manage one power cable or charge one main battery pack, making the robot more user-friendly.
* **Space Efficiency:** A high-quality buck converter is generally smaller than an entire dedicated battery pack, freeing up space in the 3-layer stack.
* **Utilizing Overhead Power:** Since the 300W PSU has plenty of overhead, it is efficient to use that existing power for the RPi 5.

### Disadvantages (Cons)
* **Voltage Instability (The Brownout Risk):** When all 7 motors accelerate simultaneously, the sudden tug on the power line can cause the 12V rail to dip. If the buck converter cannot react fast enough, the RPi 5 may trigger a low-voltage warning or crash.
* **EMI/Noise Sensitivity:** Motor noise can travel back through the shared ground line, potentially interfering with the sensitive GPIO signals or I2C/UART communication.
* **Thermal Management:** A converter capable of outputting a stable 5V 5A generates significant heat, requiring dedicated cooling or a large heatsink to prevent thermal shutdown.