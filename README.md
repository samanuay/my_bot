# my_bot — Autonomous Mobile Robot (AMR)

[![ROS 2](https://img.shields.io/badge/ROS_2-Humble_Hawksbill-22314E?style=for-the-badge&logo=ros&logoColor=white)](https://docs.ros.org/en/humble/)
[![Gazebo](https://img.shields.io/badge/Gazebo-11-F58220?style=for-the-badge&logo=gazebo&logoColor=white)](https://gazebosim.org/)
[![Nav2](https://img.shields.io/badge/Navigation-Nav2-00599C?style=for-the-badge)](https://navigation.ros.org/)
[![SLAM](https://img.shields.io/badge/Mapping-SLAM_Toolbox-10B981?style=for-the-badge)](https://github.com/SteveMacenski/slam_toolbox)
[![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](LICENSE.md)

A complete, production-grade **Differential Drive Autonomous Mobile Robot (AMR)** simulation package built for **ROS 2 Humble**. Features URDF/Xacro kinematic modeling, Gazebo physics simulation, 360° 2D LiDAR scanning, forward RGB camera streaming, real-time LiDAR-based SLAM mapping, and Nav2 autonomous waypoint navigation.

---

## Kinematics & Technical Specifications

```
                              [ LiDAR ]
                              (360° /scan)
                                  │
                       ┌──────────┴──────────┐
                       │      CHASSIS        │
                       │  (300x300x150 mm)   │
                       │       [0.5 kg]      │
                       └──────────┬──────────┘
                                  │
          ┌───────────────────────┼───────────────────────┐
          ▼                       ▼                       ▼
    [Left Wheel]           [Caster Ball]           [Right Wheel]
 (Ø66mm, Torque 200Nm)      (Frictionless)      (Ø66mm, Torque 200Nm)
```

| Parameter | Specification | Details |
| :--- | :--- | :--- |
| **Kinematic Architecture** | Two-Wheel Differential Drive | Powered rear wheels + passive low-friction front caster |
| **Wheel Diameter** | 66 mm (0.066 m) | Radius: 33 mm |
| **Track Width (Separation)**| 350 mm (0.35 m) | Distance between wheel contact centers |
| **Chassis Dimensions** | 300 × 300 × 150 mm | Rigid lightweight polycarbonate enclosure |
| **Total Weight** | ~2.5 kg | Including battery, actuators, compute & sensor payloads |
| **Max Linear Velocity** | 0.26 m/s | Tuned for safe indoor corridor traversal |
| **Max Angular Velocity** | 1.0 rad/s (~57°/s) | Agile zero-radius differential turning |
| **LiDAR Sensor** | 360° 2D Laser Scanner | 0.3m – 12.0m range, 360 samples @ 10 Hz |
| **Vision Sensor** | Forward RGB Camera | 640 × 480 @ 30 FPS, 1.089 rad FOV |
| **Target OS / ROS** | Ubuntu 22.04 LTS / ROS 2 Humble | Standard LTS deployment target |

---

## Coordinate Frame & Kinematic TF Tree

The package strictly conforms to **REP-105** (Coordinate Frames for Mobile Platforms) and **REP-103**:

```
[ map ]
   │
   ▼ (published by slam_toolbox / amcl)
[ odom ]
   │
   ▼ (published by diff_drive controller plugin)
[ base_footprint ]
   │
   ▼ (height offset to wheel axle center)
[ base_link ]
   │
   ├─► [ chassis ]
   │       ├─► [ laser_frame ]          (/scan)
   │       ├─► [ camera_link ]
   │       │       └─► [ camera_link_optical ] (/camera/image_raw)
   │       └─► [ caster_wheel ]
   │
   ├─► [ left_wheel ]
   └─► [ right_wheel ]
```

---

## Repository Structure

```
my_bot/
├── CMakeLists.txt                      # CMake build and install target definitions
├── package.xml                         # ROS 2 package manifest with execution dependencies
├── LICENSE.md                          # MIT Open Source License
├── README.md                           # Technical manual & simulation guide
│
├── description/                        # URDF / Xacro Robot Models
│   ├── robot.urdf.xacro                # Master Xacro unifying components
│   ├── robot_core.xacro                # Base link, chassis, wheels & caster
│   ├── inertial_macros.xacro           # Exact box/cylinder/sphere inertia math
│   ├── gazebo_control.xacro            # Gazebo differential drive plugin (/cmd_vel, /odom)
│   ├── lidar.xacro                     # 2D LiDAR ray sensor plugin (/scan)
│   ├── camera.xacro                    # RGB camera sensor plugin (/camera/image_raw)
│   └── ros2_control.xacro              # ros2_control hardware interface definition
│
├── config/                             # Parameter Configurations
│   ├── mapper_params_online_async.yaml # SLAM Toolbox Ceres-solver scan matching config
│   ├── nav2_params.yaml                # Nav2 stack (BT navigator, DWB planner, costmaps)
│   └── my_controllers.yaml             # ros2_control diff_drive_controller config
│
├── launch/                             # Launch Orchestration
│   ├── rsp.launch.py                   # Robot State Publisher node
│   ├── launch_sim.launch.py            # Master simulation (Gazebo + World + Robot Spawner)
│   ├── online_async_launch.py          # Real-time SLAM Toolbox mapper
│   ├── navigation.launch.py            # Nav2 autonomous navigation bringup
│   └── joystick.launch.py              # Teleop twist controller launcher
│
└── worlds/                             # Gazebo Simulation Environments
    ├── obstacles.world                 # 3D obstacle world with barriers, walls & pillars
    └── empty.world                     # Minimal ground plane world
```

---

## Quickstart & Simulation Guide

### 1. Prerequisites (ROS 2 Humble)

Ensure you have ROS 2 Humble and required simulation & navigation packages installed:

```bash
sudo apt update && sudo apt install -y \
  ros-humble-gazebo-ros-pkgs \
  ros-humble-slam-toolbox \
  ros-humble-navigation2 \
  ros-humble-nav2-bringup \
  ros-humble-xacro \
  ros-humble-teleop-twist-keyboard
```

### 2. Build the Workspace

```bash
# In your ROS 2 workspace (e.g. ~/dev_ws)
cd ~/dev_ws/src
git clone https://github.com/samanuay/my_bot.git
cd ~/dev_ws

# Build the package
colcon build --symlink-install --packages-select my_bot
source install/setup.bash
```

---

### 3. Launching the Simulation

Launch Gazebo with the custom `obstacles.world` and automatically spawn `my_bot`:

```bash
ros2 launch my_bot launch_sim.launch.py
```

### 4. Teleoperating the Robot

Open a new terminal to drive the robot around using keyboard teleop:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

---

### 5. Simultaneous Localization and Mapping (SLAM)

While the robot is driving in Gazebo, launch **SLAM Toolbox** to generate an occupancy grid map:

```bash
ros2 launch my_bot online_async_launch.py
```

Open **RViz2** to view the live mapping process:
```bash
rviz2
```
*In RViz, add **RobotModel**, **TF**, **LaserScan** (topic `/scan`), and **Map** (topic `/map`).*

Once you have explored and mapped the environment, save the map:
```bash
ros2 run nav2_map_server map_saver_cli -f ~/my_map
```

---

### 6. Autonomous Navigation (Nav2)

To enable full autonomous waypoint navigation, costmap-based obstacle avoidance, and path recovery:

```bash
ros2 launch my_bot navigation.launch.py map:=~/my_map.yaml
```

In RViz:
1. Set the **2D Pose Estimate** to initialize the AMCL particle filter at the robot's starting position.
2. Click **Nav2 Goal** anywhere in the map.
3. Watch the robot autonomously compute the global path via `NavfnPlanner`, navigate dynamic obstacles via `DWBLocalPlanner`, and come to a precise halt at the target goal.

---

## Physical Hardware Bridge

The software architecture of `my_bot` is directly transferable to a physical robot without rewriting launch or navigation files:

```
[ Raspberry Pi 4 (ROS 2 Humble) ]
       │
       ├─► RPLiDAR A1/C1 (USB Serial -> /scan)
       ├─► USB WebCam / Pi Camera (/camera/image_raw)
       │
       └─► USB Serial / UART (micro-ROS)
              │
       [ ESP32 / Arduino Motor Controller ]
              ├─► Dual H-Bridge Motor Driver (L298N / Cytron MDD10A)
              └─► Optical Wheel Encoders (Quadrature Ticks)
```

1. Replace `gazebo_control.xacro` with a real serial/micro-ROS diff-drive node publishing to `/odom` and subscribing to `/cmd_vel`.
2. Connect your physical LiDAR (e.g. RPLiDAR) to output the `sensor_msgs/LaserScan` topic to `/scan` on the `laser_frame` link.
3. Run the exact same `online_async_launch.py` and `navigation.launch.py` to achieve autonomous physical navigation.

---

## License

This project is licensed under the [MIT License](LICENSE.md).
