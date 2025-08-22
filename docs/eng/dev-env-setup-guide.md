# Complete ROS 2 Development Environment Setup Guide with Docker

## Recommended Versions (Updated August 2025)

**🎯 RECOMMENDED SETUP:**
- **Host OS**: Ubuntu 24.04 LTS (Noble Numbat) 
- **ROS 2**: Kilted Kaiju (Latest - May 2025)
- **Base Image**: `ros:kilted-desktop-full`
- **Alternative**: ROS 2 Humble (LTS) on Ubuntu 22.04 for long-term stability

## Why Docker for ROS 2 Development?

**Docker significantly simplifies ROS 2 development by providing:**

✅ **Reproducible environments** - Same setup across all team members and machines  
✅ **Isolated dependencies** - No conflicts with host system packages  
✅ **Easy deployment** - Build once, run anywhere  
✅ **Version management** - Switch between ROS 2 distributions effortlessly  
✅ **CI/CD integration** - Seamless automation and testing  
✅ **Quick setup** - Development environment ready in minutes  
✅ **Latest features** - Access to cutting-edge ROS 2 capabilities

## Prerequisites

Before starting, ensure you have:

- **Operating System**: Ubuntu 24.04 LTS (recommended), Ubuntu 22.04 LTS, Windows with WSL2, or macOS
- **Docker**: Latest version (24.0+) installed
- **Visual Studio Code**: Latest version with Dev Containers extension
- **Git**: For version control

## Method 1: Professional Template Setup (Recommended)

This approach uses Allison Thackston's production-ready template that's widely used in the ROS 2 community.

### Step 1: Initial Setup

```bash
# Clone the professional template
git clone https://github.com/athackst/vscode_ros2_workspace.git my_robot_project
cd my_robot_project

# Remove the original git history and initialize your own
rm -rf .git
git init
git add .
git commit -m "Initial commit from ROS2 workspace template"
```

### Step 2: Install Required Software

**Docker Installation:**
```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
# Log out and log back in to apply group changes

# Install Docker Compose (if not included)
sudo apt install docker-compose-plugin
```

**Visual Studio Code + Extensions:**
```bash
# Install VSCode
sudo snap install code --classic

# Required extensions (install via VSCode Extensions marketplace):
# - ms-vscode-remote.remote-containers (Dev Containers)
# - ms-iot.vscode-ros (ROS extension)
# - ms-vscode.cpptools (C++ support)
# - ms-python.python (Python support)
```

### Step 3: Configure Your Workspace

**Customize the workspace for your robot project:**

```bash
# Edit the workspace configuration
code .
```

**Key files to customize:**

1. **`.devcontainer/devcontainer.json`** - Container configuration
2. **`src/ros2.repos`** - External ROS 2 repositories to include
3. **`.vscode/settings.json`** - VSCode workspace settings

### Step 4: Launch Development Environment

When you open the folder in VSCode:

1. **VSCode will detect the devcontainer** and show a popup
2. **Click "Reopen in Container"** or press `Ctrl+Shift+P` → "Dev Containers: Reopen in Container"
3. **Wait for container build** (takes 5-10 minutes first time)
4. **Start developing!** Your terminal will show `ros@container-id:~/ws$`

## Method 2: Custom Setup for Your Robot Project

For more control over your development environment, create a custom setup.

### Step 1: Create Project Structure

```bash
# Create your robot project directory
mkdir -p ~/robot_projects/my_universal_robot
cd ~/robot_projects/my_universal_robot

# Create the standard ROS 2 workspace structure
mkdir -p src .devcontainer .vscode
```

### Step 2: Create Dockerfile

Create `.devcontainer/Dockerfile`:

```dockerfile
# Use official ROS 2 Kilted base image (latest as of August 2025)
FROM ros:kilted-desktop-full

# For long-term stability, alternative option:
# FROM ros:humble-desktop-full

# Install development tools and dependencies
RUN apt-get update && apt-get install -y \
    python3-pip \
    python3-colcon-common-extensions \
    python3-rosdep \
    python3-vcstool \
    git \
    curl \
    vim \
    gdb \
    valgrind \
    htop \
    tree \
    && rm -rf /var/lib/apt/lists/*

# Install additional ROS 2 packages for your robot (updated for Kilted)
RUN apt-get update && apt-get install -y \
    ros-kilted-navigation2 \
    ros-kilted-nav2-bringup \
    ros-kilted-robot-localization \
    ros-kilted-ros2-control \
    ros-kilted-ros2-controllers \
    ros-kilted-ublox-gps \
    ros-kilted-rplidar-ros \
    ros-kilted-v4l2-camera \
    ros-kilted-cv-bridge \
    ros-kilted-image-transport \
    && rm -rf /var/lib/apt/lists/*

# Alternative for Humble users - uncomment if using humble base image:
# RUN apt-get update && apt-get install -y \
#     ros-humble-navigation2 \
#     ros-humble-nav2-bringup \
#     ros-humble-robot-localization \
#     ros-humble-ros2-control \
#     ros-humble-ros2-controllers \
#     ros-humble-ublox-gps \
#     ros-humble-rplidar-ros \
#     ros-humble-v4l2-camera \
#     && rm -rf /var/lib/apt/lists/*

# Create a non-root user
ARG USERNAME=ros
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Set up the workspace
USER $USERNAME
WORKDIR /home/ros/ws

# Configure ROS 2 environment (updated for Kilted)
RUN echo "source /opt/ros/kilted/setup.bash" >> ~/.bashrc
RUN echo "if [ -f ~/ws/install/setup.bash ]; then source ~/ws/install/setup.bash; fi" >> ~/.bashrc

# For Humble users, use this instead:
# RUN echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
# RUN echo "if [ -f ~/ws/install/setup.bash ]; then source ~/ws/install/setup.bash; fi" >> ~/.bashrc

# Set environment variables for better performance
ENV ROS_DISTRO=kilted
ENV ROS_PYTHON_VERSION=3
ENV ROS_VERSION=2

# Set the default command
CMD ["/bin/bash"]
```

### Step 3: Create DevContainer Configuration

Create `.devcontainer/devcontainer.json`:

```json
{
    "name": "Universal Robot Control System",
    "build": {
        "dockerfile": "Dockerfile",
        "args": {
            "USERNAME": "ros"
        }
    },
    "remoteUser": "ros",
    "workspaceFolder": "/home/ros/ws",
    "workspaceMount": "source=${localWorkspaceFolder},target=/home/ros/ws,type=bind",
    
    "customizations": {
        "vscode": {
            "extensions": [
                "ms-vscode.cpptools",
                "ms-vscode.cpptools-themes",
                "ms-python.python",
                "ms-iot.vscode-ros",
                "twxs.cmake",
                "ms-vscode.cmake-tools",
                "eamodio.gitlens",
                "streetsidesoftware.code-spell-checker"
            ],
            "settings": {
                "python.defaultInterpreterPath": "/usr/bin/python3",
                "python.autoComplete.extraPaths": ["/opt/ros/kilted/lib/python3.12/site-packages"],
                "python.analysis.extraPaths": ["/opt/ros/kilted/lib/python3.12/site-packages"],
                "C_Cpp.default.includePath": [
                    "/opt/ros/kilted/include/**",
                    "/usr/include/**"
                ],
                "C_Cpp.default.compilerPath": "/usr/bin/gcc",
                "C_Cpp.default.cppStandard": "c++17",
                "C_Cpp.default.intelliSenseMode": "gcc-x64"
            }
        }
    },
    
    "containerEnv": {
        "DISPLAY": ":0",
        "ROS_DOMAIN_ID": "42",
        "ROS_LOCALHOST_ONLY": "1"
    },
    
    "runArgs": [
        "--network=host",
        "--ipc=host",
        "--pid=host",
        "--cap-add=SYS_PTRACE",
        "--security-opt=seccomp:unconfined"
    ],
    
    "mounts": [
        "source=/tmp/.X11-unix,target=/tmp/.X11-unix,type=bind",
        "source=/dev,target=/dev,type=bind"
    ],
    
    "postCreateCommand": "sudo rosdep update && rosdep install --from-paths src --ignore-src -y"
}
```

### Step 4: Create Build and Task Configuration

Create `.vscode/tasks.json`:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "colcon: build",
            "type": "shell",
            "command": "colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": []
        },
        {
            "label": "colcon: build debug",
            "type": "shell",
            "command": "colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Debug",
            "group": "build",
            "options": {
                "cwd": "${workspaceFolder}"
            }
        },
        {
            "label": "colcon: test",
            "type": "shell",
            "command": "colcon test",
            "group": "test",
            "options": {
                "cwd": "${workspaceFolder}"
            }
        },
        {
            "label": "rosdep: install dependencies",
            "type": "shell",
            "command": "rosdep install --from-paths src --ignore-src -y",
            "group": "build"
        }
    ]
}
```

### Step 5: Launch and Test

1. **Open in VSCode**: `code .`
2. **Open in container**: Press `Ctrl+Shift+P` → "Dev Containers: Open Folder in Container"
3. **Test the setup**:

```bash
# In the container terminal
ros2 pkg list  # Should show installed packages
echo $ROS_DISTRO  # Should output "kilted" (or "humble" if using Humble)

# Test basic functionality
ros2 run demo_nodes_cpp talker &
ros2 run demo_nodes_cpp listener
```

## Method 3: Docker Compose for Complex Setups

For multi-service development (robot + simulation + monitoring), use Docker Compose.

### Step 1: Create docker-compose.yml

```yaml
version: '3.8'

services:
  robot_dev:
    build:
      context: .
      dockerfile: .devcontainer/Dockerfile
    container_name: robot_development
    network_mode: host
    ipc: host
    privileged: true
    environment:
      - DISPLAY=:0
      - ROS_DOMAIN_ID=42
    volumes:
      - .:/home/ros/ws:rw
      - /tmp/.X11-unix:/tmp/.X11-unix:rw
      - /dev:/dev:rw
      - ~/.ssh:/home/ros/.ssh:ro
    command: tail -f /dev/null
    
  gazebo_sim:
    image: ros:kilted-desktop-full
    container_name: gazebo_simulation
    network_mode: host
    environment:
      - DISPLAY=:0
      - ROS_DOMAIN_ID=42
    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix:rw
    command: ros2 launch gazebo_ros gazebo.launch.py
    depends_on:
      - robot_dev
```

### Step 2: Launch Multi-Service Environment

```bash
# Start all services
docker-compose up -d

# Attach to development container
docker exec -it robot_development bash

# In separate terminals, you can access different services
docker exec -it gazebo_simulation bash
```

## Hardware Integration

### USB Device Access (GPS, LiDAR, Camera)

Add to your `devcontainer.json` runArgs:

```json
"runArgs": [
    "--device=/dev/ttyUSB0:/dev/ttyUSB0",  // GPS device
    "--device=/dev/ttyACM0:/dev/ttyACM0",  // Alternative GPS port
    "--device=/dev/video0:/dev/video0",    // Camera
    "--volume=/dev:/dev:rw"                // All devices (less secure)
]
```

### GPU Access (for Computer Vision)

For NVIDIA GPUs:

```json
"runArgs": [
    "--gpus=all",
    "--runtime=nvidia"
]
```

## Development Workflow

### Daily Development Process

1. **Start your environment**:
   ```bash
   code ~/robot_projects/my_universal_robot
   # VSCode opens in container automatically
   ```

2. **Install new dependencies**:
   ```bash
   # Add to package.xml, then:
   rosdep install --from-paths src --ignore-src -y
   ```

3. **Build and test**:
   ```bash
   # Use VSCode tasks: Ctrl+Shift+P → "Tasks: Run Task"
   # Or manually:
   colcon build --symlink-install
   source install/setup.bash
   ```

4. **Version control**:
   ```bash
   git add .
   git commit -m "Added GPS integration"
   git push origin main
   ```

### Best Practices

**✅ Do:**
- Use volume mounts for source code (persists changes)
- Mount only necessary devices for security
- Use multi-stage builds for production images
- Keep Dockerfiles in version control
- Use .dockerignore to exclude build artifacts

**❌ Don't:**
- Install packages directly in running containers (use Dockerfile)
- Mount entire /dev directory in production
- Store secrets in Docker images
- Run containers as root in production

## Troubleshooting Common Issues

### GUI Applications (RViz, Gazebo)

**Problem**: "Cannot connect to X server"

**Solution**:
```bash
# On host (Ubuntu):
xhost +local:docker

# In devcontainer.json, ensure:
"containerEnv": {
    "DISPLAY": ":0"
},
"mounts": [
    "source=/tmp/.X11-unix,target=/tmp/.X11-unix,type=bind"
]
```

### USB Device Not Found

**Problem**: Device permissions denied

**Solution**:
```bash
# Add user to dialout group in Dockerfile:
RUN usermod -a -G dialout $USERNAME

# Or mount with proper permissions:
"--device=/dev/ttyUSB0:/dev/ttyUSB0:rwm"
```

### Slow Performance

**Problem**: Container feels sluggish

**Solutions**:
- Use `--ipc=host` for faster shared memory
- Mount build directories as volumes
- Use Docker BuildKit for faster builds
- Increase Docker desktop memory allocation

## Production Deployment

### Creating Deployment Images

```dockerfile
# Multi-stage build for production (updated for Kilted)
FROM ros:kilted-desktop-full as development
# ... development setup ...

FROM ros:kilted-ros-base as production
COPY --from=development /home/ros/ws/install /opt/robot_ws/install
RUN echo "source /opt/robot_ws/install/setup.bash" >> ~/.bashrc
ENTRYPOINT ["ros2", "launch", "robot_bringup", "robot.launch.py"]
```

### Robot Deployment

```bash
# Build production image
docker build --target production -t my-robot:v1.0 .

# Deploy to robot
docker save my-robot:v1.0 | ssh robot-pi docker load
ssh robot-pi "docker run -d --restart=unless-stopped my-robot:v1.0"
```

## Version Selection Guide

### **ROS 2 Kilted Kaiju (Latest - Recommended for New Projects)**
- **Release Date**: May 2025
- **Ubuntu Support**: 24.04 LTS (Noble)
- **Support Until**: November 2026
- **Best For**: New projects, latest features, cutting-edge development
- **Key Features**: 
  - Eclipse Zenoh middleware support
  - 10x Python performance improvements
  - Enhanced ROSBag capabilities
  - NV12 image format support
  - Better Windows 11 support

### **ROS 2 Humble Hawksbill (LTS - Recommended for Production)**
- **Release Date**: May 2022
- **Ubuntu Support**: 22.04 LTS (Jammy) 
- **Support Until**: May 2027
- **Best For**: Production systems, long-term stability
- **Key Features**: 
  - 5-year LTS support
  - Proven stability
  - Extensive package ecosystem
  - Well-documented and tested

### **Which Should You Choose?**

**For your 6-month universal robot project:**
- **Use Kilted Kaiju** if you want latest features and don't mind occasional updates
- **Use Humble** if you prioritize stability and long-term support

## Next Steps

1. **Start with Method 1** (professional template) for quick setup
2. **Customize as needed** using Method 2 concepts
3. **Add your robot packages** to the src/ directory
4. **Configure hardware interfaces** using the hardware integration guide
5. **Set up CI/CD** using the included GitHub Actions workflow

This Docker-based development environment will provide you with a robust, reproducible setup for your universal robot control system development!