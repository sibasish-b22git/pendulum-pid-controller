# ROS 2 Custom PID Pendulum Controller

This repository contains a ROS 2 (Jazzy) and Gazebo Harmonic simulation of a 1-DOF inverted pendulum, controlled by a custom-written Python PID controller. 

Instead of relying entirely on standard `ros2_controllers`, this project demonstrates how to bridge custom control theory math with the `ros2_control` framework. The Python node reads live physical sensor data from Gazebo, calculates the Proportional-Integral-Derivative (PID) corrections at 100Hz, and sends smooth trajectory commands to the simulated hardware.

## 🛠️ Tech Stack & Requirements
* **OS:** Ubuntu 24.04
* **ROS 2:** Jazzy Jalisco
* **Simulator:** Gazebo Harmonic
* **Dependencies:** ```bash
  sudo apt install ros-jazzy-ros2-control ros-jazzy-ros2-controllers ros-jazzy-ros-gz-sim ros-jazzy-xacro
  ```

## 📦 Package Architecture

This project is divided into two primary packages:

1. **`pendulum_control_description` (C++ / CMake)**
   * Contains the physical blueprints (`.xacro` / `.urdf`).
   * Configures the `gz_ros2_control` hardware abstraction plugins.
   * Houses the primary launch file that sequences Gazebo, the `robot_state_publisher`, and the `joint_state_broadcaster` using Event Handlers to prevent race conditions.

2. **`pid_controller` (Python)**
   * Contains the custom `PID_controller` Python node.
   * Dynamically subscribes to `/joint_states` and targets `/joint_setpoint`.
   * Calculates real-time positional error using actual sensor feedback.
   * Configurable via ROS parameters (`kp`, `ki`, `kd`, `joint_name`).

## 🚀 Installation & Build

Clone this repository into your ROS 2 workspace `src` folder:

```bash
cd ~/your_ros2_ws/src
git clone <your-repository-url>
```

Build the packages and source the installation:

```bash
cd ~/your_ros2_ws
colcon build --packages-select pendulum_control_description pid_controller
source install/setup.bash
```

## 🎮 Usage Instructions

### 1. Launch the Simulation Environment
This command boots up Gazebo Harmonic, compiles the Xacro blueprint, and spawns the pendulum with its virtual encoders active.
```bash
ros2 launch pendulum_control_description pendulum_controller.launch.py
```

### 2. Launch the Custom PID Controller
In a new terminal (remember to `source install/setup.bash`), boot up the Python brain. It will automatically load the YAML configuration and wait for a command.
```bash
ros2 launch pid_controller pid_controller.launch.py
```
*(Note: Ensure the `joint_name` parameter in your `pid.yaml` exactly matches `revolute_joint` from the URDF).*

### 3. Send a Target Setpoint
In a third terminal, publish a target angle (in radians) to the controller. Use the `-1` flag to send it just once.
```bash
ros2 topic pub -1 /joint_setpoint std_msgs/msg/Float64 "{data: 1.5}"
```
You should see the pendulum swing to the target position, and the PID controller terminal will log the `Target`, `Error`, and `Command` outputs at 100Hz.

## 📈 Visualizing the Controller (Optional)
To watch the PID controller's step response and damping in real-time, you can graph the joint states:
```bash
ros2 run rqt_plot rqt_plot
```
In the GUI, add `/joint_states/position[0]` and `/joint_states/velocity[0]` to watch the physical response curve as you send different setpoints.

---
