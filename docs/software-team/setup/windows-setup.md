# Windows / Linux Set Up
## 1. Prerequisites

Before cloning the repository, ensure your Windows host machine has the following tools installed:

1. **WSL 2 (Windows Subsystem for Linux):** Install a Debian or Ubuntu distribution.
    - https://learn.microsoft.com/en-us/windows/wsl/install
2. **Docker Desktop for Windows:** During installation, ensure **"Use WSL 2 instead of Hyper-V"** is checked.
    - https://docs.docker.com/desktop/setup/install/windows-install/
    -    Open Docker Settings > Resources > WSL Integration, and enable integration for your specific Linux distro.
3. **Visual Studio Code:** Install the **Dev Containers** extension by Microsoft.
4. **Python (Anaconda Recommended):** Ensure Python is installed natively on your Windows system to run the webcam capture script.

## 2. Clone into the Linux Filesystem

> **NOTE:** Do not clone this repository into your standard Windows `C:\Users\...` directory.

You must clone the code directly into the WSL filesystem.

1. Open your WSL Terminal (Debian/Ubuntu).
2. Create a workspace and clone the repository:
   ```bash
   mkdir -p ~/workspace
   cd ~/workspace
   git clone https://github.com/sfu-akcse/Aether.git
   cd Aether
   ```
3. Launch VS Code directly from the WSL terminal
    ```bash
    code .
    ```
## 3. Launch the Dev Container

1. Once VS Code opens, press **Ctrl+Shift+P** to open the Command Palette.
2. Select **Dev Containers: Reopen in Container**.
3. Docker will begin building the environment. (Note: The first build will take several minutes to download the OS image).
4. Once loaded, open a new terminal in VS Code. You are now running as 'root' inside the container.

### Troubleshooting

Paste the following commands in the VS Code terminal to ensure all dependencies are working as intended.
```bash
ros2 topic list
python3 -c "import cv2; import mediapipe; print('CV and ML successfully loaded!')"
cmake --version && gcc --version
```

The expected output:
```bash
/parameter_events
/rosout
CV and ML successfully loaded!
cmake version 3.22.1
```

## 4. Hardware & Webcam Setup

Docker containers cannot natively access Windows USB devices. To provide the container with computer vision data, we run the capture script natively on Windows and stream it over the local network bridge.

1. Open a **Native Windows Terminal** (PowerShell or Anaconda Prompt). Do not use the VS Code Dev Container terminal for this step.

2. Ensure you have the required Python libraries installed on Windows:
    ```bash
    pip install opencv-python mediapipe
    ```
3. Run the streaming script by pointing Python to the network path of your WSL files. (Replace \<your_wsl_username> with your actual Linux username):
    ```bash
    python \\wsl.localhost\Debian\home\<your_wsl_username>\akcse\Aether\scripts\host_webcam_stream.py --port 8080
    ```
4. Your physical webcam light should turn on. The camera feed is now broadcasting to localhost:8080 for your devcontainer to connect to.
5. Add host gateway mapping in `.devcontainer/devcontainer.json`:
    ```json
    "runArgs": [
        "--ipc=host",
        "--add-host=host.docker.internal:host-gateway"
    ]
    ```
6. Rebuild/reopen the devcontainer.
7. In container, run:
    ```bash
    export CAMERA_SOURCE=http://host.docker.internal:8080/video.mjpg
    python3 src/main.py
    ```