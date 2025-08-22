🎨 Legend:

- 🔵 Repository Level (Light Blue): Top-level directories and infrastructure
- 🟣 Package Level (Light Purple): ROS 2 packages organized by domain
- 🟢 Node Level (Light Green): Executable ROS 2 nodes
- 🟠 Configuration Level (Light Orange): Configuration files and parameters
- 🔴 Launch Level (Light Pink): Launch files for system startup

📋 Key Organizational Principles Shown:
1. Repository Structure:

Infrastructure files (Docker, CI/CD, VSCode) at the root
Clear separation between packages (src/) and global configs (config/, launch/)

2. Package Organization:

Domain-based grouping (hardware, navigation, perception, control)
Single-responsibility packages with clear boundaries
Shared interfaces package for inter-package communication

3. Node Hierarchy:

Hardware nodes publish sensor data and control actuators
Navigation nodes handle path planning and localization
Perception nodes process camera and sensor data
Control nodes manage robot behavior and state

4. Dependency Flow:

External dependencies feed into hardware drivers
Hardware provides data to perception and navigation
Control coordinates high-level behavior
Bringup orchestrates the entire system

5. Configuration Management:

Domain-specific config directories mirror package structure
Launch files provide different system startup modes
Global configs separate from package-specific settings

```mermaid
graph LR
    %% Repository Level
    Root[my_robot_project<br/>📁 Monorepo Root] --> DevEnv[.devcontainer/<br/>🐳 Docker Dev Environment]
    Root --> CI[.github/<br/>⚙️ CI/CD Workflows]
    Root --> VSCode[.vscode/<br/>🔧 IDE Configuration]
    Root --> Docs[docs/<br/>📚 Documentation]
    Root --> Src[src/<br/>📦 ROS 2 Packages]
    Root --> GlobalConfig[config/<br/>⚙️ Global Configuration]
    Root --> Launch[launch/<br/>🚀 System Launch Files]
    
    %% Package Level Organization
    Src --> Bringup[robot_bringup/<br/>🎯 System Integration]
    Src --> Description[robot_description/<br/>🤖 Robot Model]
    Src --> Hardware[robot_hardware/<br/>🔌 Hardware Drivers]
    Src --> Navigation[robot_navigation/<br/>🗺️ Navigation Stack]
    Src --> Perception[robot_perception/<br/>👁️ Computer Vision]
    Src --> Control[robot_control/<br/>🎮 Behavioral Control]
    Src --> Interfaces[robot_interfaces/<br/>📨 Custom Messages]
    Src --> External[external/<br/>📥 Third-party Packages]
    
    %% Hardware Package Breakdown
    Hardware --> HWSensors[sensors/<br/>📡 Sensor Drivers]
    Hardware --> HWActuators[actuators/<br/>⚡ Motor Controllers]
    
    %% Hardware Nodes
    HWSensors --> GPSNode[gps_driver_node<br/>🛰️ GPS Data Publisher]
    HWSensors --> LiDARNode[lidar_driver_node<br/>📊 Point Cloud Publisher]
    HWSensors --> CameraNode[camera_driver_node<br/>📷 Image Publisher]
    
    HWActuators --> MotorNode[motor_controller_node<br/>🔧 Wheel Control]
    HWActuators --> ServoNode[servo_driver_node<br/>📐 Servo Control]
    
    %% Navigation Package Breakdown
    Navigation --> NavNodes[nav_nodes/<br/>🧭 Custom Navigation]
    Navigation --> NavConfig[nav_config/<br/>📋 Nav2 Parameters]
    
    NavNodes --> PathPlannerNode[path_planner_node<br/>🛤️ Custom Path Planning]
    NavNodes --> LocalizationNode[localization_node<br/>📍 Robot Positioning]
    
    %% Perception Package Breakdown
    Perception --> VisionNodes[vision_nodes/<br/>👀 Computer Vision]
    Perception --> SensorFusion[sensor_fusion/<br/>🔀 Data Integration]
    
    VisionNodes --> ObjectDetectionNode[object_detection_node<br/>🎯 Object Recognition]
    VisionNodes --> ImageProcessorNode[image_processor_node<br/>🖼️ Image Processing]
    
    %% Control Package Breakdown
    Control --> ControlNodes[control_nodes/<br/>🎛️ Behavior Control]
    ControlNodes --> StateMachineNode[state_machine_node<br/>🔄 Robot State Management]
    ControlNodes --> TaskPlannerNode[task_planner_node<br/>📝 Mission Planning]
    
    %% Interfaces Package
    Interfaces --> CustomMsgs[custom_msgs/<br/>💬 Message Definitions]
    Interfaces --> CustomSrvs[custom_srvs/<br/>🔧 Service Definitions]
    Interfaces --> CustomActions[custom_actions/<br/>⏳ Action Definitions]
    
    %% Bringup Package
    Bringup --> SystemLaunch[system_launch/<br/>🚀 Main Launch Files]
    Bringup --> SystemConfig[system_config/<br/>⚙️ Robot-wide Settings]
    
    SystemLaunch --> FullRobotLaunch[robot.launch.py<br/>🤖 Complete System]
    SystemLaunch --> HardwareLaunch[hardware.launch.py<br/>🔌 Hardware Only]
    SystemLaunch --> SimulationLaunch[simulation.launch.py<br/>🌐 Gazebo Simulation]
    
    %% Description Package
    Description --> URDF[robot.urdf.xacro<br/>🦴 Robot Structure]
    Description --> Meshes[meshes/<br/>🗿 3D Models]
    Description --> Materials[materials/<br/>🎨 Textures & Colors]
    
    %% Configuration Files
    GlobalConfig --> HWConfig[hardware/<br/>🔧 Hardware Settings]
    GlobalConfig --> NavConfig2[navigation/<br/>🗺️ Navigation Params]
    GlobalConfig --> PercConfig[perception/<br/>👁️ Vision Settings]
    
    %% External Dependencies
    External --> Nav2[navigation2/<br/>🧭 ROS Navigation Stack]
    External --> VisionOpenCV[vision_opencv/<br/>📷 OpenCV Integration]
    External --> UbloxGPS[ublox_gps/<br/>🛰️ GPS Driver]
    
    %% Styling
    classDef repoLevel fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef packageLevel fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef nodeLevel fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef configLevel fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef launchLevel fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    
    class Root,DevEnv,CI,VSCode,Docs,GlobalConfig,Launch repoLevel
    class Src,Bringup,Description,Hardware,Navigation,Perception,Control,Interfaces,External packageLevel
    class GPSNode,LiDARNode,CameraNode,MotorNode,ServoNode,PathPlannerNode,LocalizationNode,ObjectDetectionNode,ImageProcessorNode,StateMachineNode,TaskPlannerNode nodeLevel
    class HWConfig,NavConfig2,PercConfig,SystemConfig,NavConfig configLevel
    class SystemLaunch,FullRobotLaunch,HardwareLaunch,SimulationLaunch launchLevel
    ```