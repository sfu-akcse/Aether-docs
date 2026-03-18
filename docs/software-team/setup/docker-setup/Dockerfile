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