# Raspberry Pi 5 or PC configuration for Bus Servo (Adapter/Driver)
## WIRING
For all methods listed below, you can power the [servo driver board](https://www.waveshare.com/bus-servo-adapter-a.htm) using a 9-12.6V power source via a 5.5*2.1mm DC connector or the 5.08mm screw terminals. 

### Method 1: USB Connection (Recommended for Raspberry Pi 5)
This is the easiest method and frees up your GPIO pins.


>1. Connect the USB Type-C port on the adapter to a standard USB port on the Pi 5.
>2. Move the control mode switch jumper on the adapter board to position B.
>3. The Raspberry Pi will automatically mount the device using its built-in USB-to-serial kernel drivers. It will typically show up as /dev/ttyUSB0.


### Method 2: UART Connection (GPIO Pins)
If you need your USB ports for other devices, you can wire it directly to the Pi's GPIO header.


>1. Connect the adapter's RX to the Pi's TX (GPIO 14 / Pin 8).
>2. Connect the adapter's TX to the Pi's RX (GPIO 15 / Pin 10).
>3. Connect a common Ground.
>4. Move the control mode switch jumper on the adapter board to position A

### Method 3: PC USB Connection

>1. Connect the USB Type-C port on the adapter to a standard USB port on your PC.
>2. Move the control mode switch jumper on the adapter board to position B.
>3. Open your device manager and check under 'Port (COM & LPT)' for a new device.

## FIRMWARE
 ### OS Configuration (For GPIO UART Only)
If you choose the USB method, you can skip this step. If you wired the adapter to the GPIO pins, you need to enable the serial hardware on the Raspberry Pi:
>1. Open your terminal and run: sudo raspi-config
>2. Navigate to Interface Options -> Serial Port.
>3. When asked "Would you like a login shell to be accessible over serial?", select No.
>4. When asked "Would you like the serial port hardware to be enabled?", select Yes.
>5. Save, exit, and reboot your Pi. The port will typically map to /dev/serial0.
>6. Installing the Software "Drivers" (SDK & Libraries)
>7. To actually move the servos, you need the Python environment configured.

### Raspberry Pi Set-up
#### 1. Install PySerial
>This library allows Python to read and write to the serial ports.

1. Open the Pi terminal (the black `>_` icon) and run the following commands:

```
sudo apt update
sudo apt install python3-serial
```

(Alternatively, you can use a virtual environment and run `pip install pyserial`)

>**Common Bug:** On newer versions of Raspberry Pi OS (Bookworm), you might get an error saying the environment is "externally managed."\
>**The Fix:** If the standard command fails, you simply add --break-system-packages to the end of the install command or use a virtual environment.

2. Once the installation finishes, you can verify it's working by typing:
```
python3 -c "import serial; print(serial.__version__)"
```
If it prints a version number (like 3.5), the "driver" is officially ready to talk to your Waveshare adapter.

#### 2. Download the Waveshare SDK
Waveshare provides the actual "driver" code as a downloadable Python library.
Navigate to the official [Waveshare Wiki page for the Bus Servo Adapter (A)](https://www.waveshare.com/wiki/Bus_Servo_Adapter_%28A%29#Introduction).

1. Scroll to the "[ST Series Python Example](https://www.waveshare.com/wiki/Bus_Servo_Adapter_%28A%29#ST_Series_Servo_Python_Example)" section and [download](https://files.waveshare.com/wiki/Bus-Servo-Adapter-(A)/STServo_Python.zip) the ST/SC serial bus servo control library (Python).

2. Extract the ZIP file on your Raspberry Pi.

Inside that folder, you will find example scripts (like ping.py or writePos.py) and the core libraries that handle the ST and SC servo protocols.
>**Note on Power:** The Raspberry Pi 5 cannot power bus servos through the USB or GPIO data connections. You must provide a separate 9V to 12.6V DC power supply (matching your specific servos' requirements) directly to the adapter's barrel jack or screw terminals.

#### 3. Run Example Scripts
1. Inside the folder, you will see example files like ping.py (to check if the servo is responding) or writePos.py (to make it move). Before you run them, you need to open the script in a text editor (like nano or Thonny) and tell it how you connected the board.

2. Look near the top of the script for a line that defines the serial port. Change it to match your physical setup:

If you used the USB cable: Change it to `/dev/ttyUSB0`

If you used the GPIO pins: Change it to `/dev/serial0`

### Windows PC Set-up
#### 1. Install Python
1. Download standalone installer for Python on the [official Python website](https://www.python.org/downloads/).
2. After downloading, double-click `python-3.14.3-amd64.exe` to install, and click "Customize installation" to enter "Optional Features". Keep clicking on "Next" to enter the "Advanced Options" interface. Be sure to keep "Add Python to environment variables" checked.
#### 2. Download the Waveshare SDK
Waveshare provides the actual "driver" code as a downloadable Python library.
Navigate to the official [Waveshare Wiki page for the Bus Servo Adapter (A)](https://www.waveshare.com/wiki/Bus_Servo_Adapter_%28A%29#Introduction).

1. Scroll to the "[ST Series Python Example](https://www.waveshare.com/wiki/Bus_Servo_Adapter_%28A%29#ST_Series_Servo_Python_Example)" section and [download](https://files.waveshare.com/wiki/Bus-Servo-Adapter-(A)/STServo_Python.zip) the ST/SC serial bus servo control library (Python).
2. Extract the ZIP file.
3. Open the extracted file and, in the windows file explorer, copy the address bar.
4. Press Windows Key + R and type in `cmd`, then hit enter. Alternatively, you can just open the windows command terminal.
5. Use the change directory command `cd` to navigate to the folder.
```
cd C:\Users\alexs\Downloads\STServo_Python\STServo_Python
```
6. Run the following command:
```
stservo-env\Scripts\activate.bat
```
7. Run the following command:
```
python -m pip install -r requirements.txt
```
#### 3. Connecting to Servos
Before running any example scripts, you need to direct the script to your servo driver.
1. Open up 'Device Manager' and look for your device's port number under 'Ports (COM & LPT)'.\
(e.g. The port number should look something like "COM41")
2. Open the script you want to run (e.g. ping.py) on a code editor and change the `DEVICENAME` so that it looks like this:
```
DEVICENAME = 'COM41'
```
3. Going back to your windows terminal, you can now run a script. With 'ping.py' as an example, enter:
```
python ping.py
```
> These instructions were taken from the [Serial Bus wikipage](https://www.waveshare.com/wiki/Bus_Servo_Adapter_%28A%29#ST_Series_Servo_Python_Example) where more instructions and Python examples can be found.