# Servo Driver Research

Instead of manually connecting each servos to our Raspberry pi, it is more ideal to use a servo driver for 3 main reasons:

1. Power Management: Microcontrollers (Arduino) and single-board computers (Raspberry Pi) cannot provide enough current to drive multiple servos simultaneously. Connecting them directly risks voltage drops that can cause your Pi to reset or even permanently damage its GPIO pins due to high stall currents.

2. Precision and Timing: A Raspberry Pi runs an operating system (Linux) that isn't "real-time," meaning it can have slight delays (jitter) when generating the PWM signals needed for smooth movement. A dedicated driver board has its own internal clock to provide perfectly steady signals.

3. Pin Efficiency: Instead of using one pin on your Pi/Arduino for every single servo, a driver typically uses an I2C interface, allowing you to control up to 16 servos using only two pins. 

Typical servo driver used in robotic projects is the Adafruit PCA9685 16-Channel Driver. Easy cable management to connect to each servos, require 5-6 V power input that will directly connect to the channels that deliver power to the motors.


#### The Limits of a Servo Driver Board

1. Current Capacity: Most PCA9685 breakout boards are designed to handle about 8A to 10A total across all channels.

2. High-Torque Demand: A single high-torque servo can draw 1A to 2.5A when moving a heavy load or stalling.

3. Voltage Limit: Most of these boards are rated for a maximum of 6V on the power input. Exceeding this can damage the board or the servos themselves.

Further research on the motors we will use are need to find out if using a PCA9685-based boards will be an issue. 
