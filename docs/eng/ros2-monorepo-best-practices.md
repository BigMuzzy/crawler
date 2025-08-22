# ROS 2 Monorepo Organization Best Practices for Small Teams

## Recommended Directory Structure

```
my_robot_project/
├── .devcontainer/           # Docker development environment
├── .github/                 # CI/CD workflows
├── .vscode/                 # VS Code settings
├── docs/                    # Documentation
├── src/                     # ROS 2 packages organized by domain
│   ├── robot_bringup/       # Main launch and configuration
│   ├── robot_description/   # URDF, meshes, robot model
│   ├── robot_hardware/      # Hardware interfaces & drivers
│   │   ├── sensors/         # Sensor drivers (GPS, LiDAR, cameras)
│   │   └── actuators/       # Motor controllers, servo drivers
│   ├── robot_navigation/    # Navigation stack configuration
│   ├── robot_perception/    # Computer vision, sensor processing
│   ├── robot_control/       # High-level control logic
│   ├── robot_interfaces/    # Custom messages, services, actions
│   └── external/            # Third-party packages (if needed)
├── config/                  # Global configuration files
├── launch/                  # System-wide launch files
├── scripts/                 # Utility scripts
├── tests/                   # Integration tests
└── tools/                   # Development tools
```

## Core Package Organization Principles

### 1. **Domain-Based Separation**
Organize packages by functional domains rather than technical layers:

```
✅ Good: robot_navigation/, robot_perception/, robot_hardware/
❌ Avoid: cpp_packages/, python_packages/, drivers/
```

### 2. **Single Responsibility Packages**
Each package should have one clear purpose:

- **robot_bringup**: System launch files, main robot startup
- **robot_description**: URDF, mesh files, robot model definition
- **robot_hardware**: All hardware interfaces and device drivers
- **robot_interfaces**: Custom message/service/action definitions
- **robot_navigation**: Navigation configuration and custom nav nodes
- **robot_perception**: Computer vision, sensor fusion, perception algorithms
- **robot_control**: High-level behavioral control, state machines

### 3. **Consistent Naming Convention**

```bash
# Package naming: {robot_name}_{domain}
my_robot_bringup
my_robot_navigation
my_robot_perception

# Node naming: {domain}_{function}_node
gps_driver_node
camera_processor_node
path_planner_node

# Topic naming: /{robot_name}/{domain}/{data_type}
/my_robot/sensors/gps
/my_robot/camera/image_raw
/my_robot/navigation/cmd_vel
```

## Package Structure Template

### Standard Package Layout
```
robot_hardware/
├── package.xml              # Package dependencies and metadata
├── CMakeLists.txt          # Build configuration
├── README.md               # Package documentation
├── src/                    # Source code
│   ├── nodes/              # Executable nodes
│   ├── drivers/            # Hardware driver classes
│   └── utils/              # Utility functions
├── include/robot_hardware/ # Header files (C++)
├── launch/                 # Launch files specific to this package
├── config/                 # Configuration files (YAML, params)
├── test/                   # Unit tests
└── scripts/                # Python scripts/tools
```

## Dependency Management Strategy

### 1. **Internal Dependencies**
Keep dependencies flowing in one direction to avoid circular dependencies:

```
robot_bringup → robot_navigation → robot_control → robot_hardware
             → robot_perception → robot_interfaces
```

### 2. **External Dependencies**
Use the workspace `ros2.repos` file for external ROS packages:

```yaml
# src/ros2.repos
repositories:
  # Navigation stack
  navigation2:
    type: git
    url: https://github.com/ros-planning/navigation2.git
    version: kilted
  
  # Hardware drivers
  ublox_gps:
    type: git
    url: https://github.com/KumarRobotics/ublox.git
    version: ros2
  
  # Computer vision
  vision_opencv:
    type: git
    url: https://github.com/ros-perception/vision_opencv.git
    version: kilted
```

### 3. **Package Dependencies**
Define clear dependency levels in `package.xml`:

```xml
<!-- robot_hardware/package.xml -->
<package format="3">
  <name>robot_hardware</name>
  
  <!-- Build dependencies -->
  <build_depend>rclcpp</build_depend>
  <build_depend>robot_interfaces</build_depend>
  
  <!-- Runtime dependencies -->
  <exec_depend>rclcpp</exec_depend>
  <exec_depend>robot_interfaces</exec_depend>
  
  <!-- Test dependencies -->
  <test_depend>gtest</test_depend>
</package>
```

## Configuration Management

### 1. **Hierarchical Configuration**
```
config/
├── robot.yaml              # Robot-wide parameters
├── hardware/
│   ├── sensors.yaml        # Sensor configurations
│   └── actuators.yaml      # Motor/servo settings
├── navigation/
│   ├── nav2_params.yaml    # Navigation parameters
│   └── costmap_params.yaml # Costmap settings
└── perception/
    └── camera_params.yaml  # Camera calibration
```

### 2. **Environment-Specific Configs**
```
config/
├── environments/
│   ├── development.yaml    # Dev environment overrides
│   ├── simulation.yaml     # Gazebo simulation settings
│   └── production.yaml     # Robot deployment settings
```

## Launch File Organization

### 1. **Hierarchical Launch Structure**
```
launch/
├── robot.launch.py         # Main system launch (everything)
├── hardware.launch.py      # Hardware drivers only
├── navigation.launch.py    # Navigation stack
├── perception.launch.py    # Sensors and perception
└── simulation.launch.py    # Gazebo simulation
```

### 2. **Modular Launch Files**
```python
# launch/robot.launch.py
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource

def generate_launch_description():
    return LaunchDescription([
        IncludeLaunchDescription(
            PythonLaunchDescriptionSource([
                get_package_share_directory('robot_hardware'),
                '/launch/hardware.launch.py'
            ])
        ),
        IncludeLaunchDescription(
            PythonLaunchDescriptionSource([
                get_package_share_directory('robot_navigation'),
                '/launch/navigation.launch.py'
            ])
        ),
    ])
```

## Development Workflow Best Practices

### 1. **Branch Strategy for Small Teams**
```
main                    # Stable, deployable code
├── develop            # Integration branch
├── feature/gps-driver # Feature branches
├── feature/navigation
└── hotfix/sensor-bug  # Critical fixes
```

### 2. **Build and Test Strategy**
```bash
# Local development cycle
colcon build --packages-select robot_hardware  # Build single package
colcon test --packages-select robot_hardware   # Test single package

# Full system build
colcon build --symlink-install                 # Build all packages
colcon test                                     # Run all tests
```

### 3. **Continuous Integration**
The provided `.github/workflows/ros.yaml` already includes:
- Linting (cppcheck, cpplint, flake8, etc.)
- Building all packages
- Running tests

## Hardware Integration Patterns

### 1. **Driver Package Structure**
```
robot_hardware/
├── src/
│   ├── nodes/
│   │   ├── gps_driver_node.cpp
│   │   ├── lidar_driver_node.cpp
│   │   └── camera_driver_node.cpp
│   └── drivers/
│       ├── gps_driver.cpp
│       ├── lidar_driver.cpp
│       └── camera_driver.cpp
└── launch/
    ├── gps.launch.py
    ├── lidar.launch.py
    └── camera.launch.py
```

### 2. **Hardware Abstraction**
```cpp
// include/robot_hardware/sensor_interface.hpp
class SensorInterface {
public:
    virtual bool initialize() = 0;
    virtual bool read_data() = 0;
    virtual void publish_data() = 0;
};

// Specific implementations
class GPSDriver : public SensorInterface { ... };
class LiDARDriver : public SensorInterface { ... };
```

## Testing Strategy

### 1. **Test Organization**
```
tests/
├── unit/               # Unit tests per package
├── integration/        # Cross-package integration tests
├── simulation/         # Gazebo-based tests
└── hardware/           # Hardware-in-the-loop tests
```

### 2. **Test Types**
```bash
# Unit tests (fast, isolated)
colcon test --packages-select robot_hardware

# Integration tests (slower, multiple packages)
colcon test --packages-select robot_bringup

# System tests (full robot simulation)
ros2 launch robot_bringup simulation.launch.py
```

## Documentation Strategy

### 1. **Multi-Level Documentation**
```
docs/
├── README.md           # Quick start guide
├── architecture.md     # System architecture
├── deployment.md       # Deployment instructions
├── api/               # Auto-generated API docs
└── tutorials/         # Step-by-step guides
```

### 2. **Package-Level Documentation**
Each package should have:
- **README.md**: Purpose, usage, dependencies
- **API documentation**: Auto-generated from code comments
- **Configuration guide**: Parameter explanations

## Small Team Specific Recommendations

### 1. **Keep It Simple**
- Start with fewer packages, split when they become complex
- Prefer composition over inheritance
- Use standard ROS 2 patterns and tools

### 2. **Shared Ownership**
- Everyone should understand the overall architecture
- Rotate code reviews across team members
- Document decisions and rationale

### 3. **Iterative Development**
- Start with basic functionality, add complexity gradually
- Use feature flags for experimental features
- Maintain working main branch at all times

### 4. **Communication**
- Use descriptive commit messages
- Document breaking changes in PR descriptions
- Regular architecture discussions

## Tools and Automation

### 1. **Development Tools**
```bash
# Code formatting
ament_uncrustify --reformat src/
ament_autopep8 --reformat src/

# Dependency analysis
colcon graph --dot | dot -Tpng -o deps.png

# Package creation
ros2 pkg create --build-type ament_cmake robot_hardware
```

### 2. **Quality Gates**
- All code must pass linting
- All tests must pass
- Code review required for main branch
- Documentation updated for new features

This structure provides a solid foundation that can evolve with your robot's complexity while maintaining clarity and organization for your small team.