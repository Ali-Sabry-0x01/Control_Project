# ARL Autonomous Vehicle Control Track

**Autotronics Research Lab (ARL) — Ain Shams University**  
*Course: Autonomous Vehicles & Drive-by-Wire Systems | Individual Project*

**Author:** Ali Sabry Ibraheem  
**Student ID:** 2400546  
**Department:** Computer and Systems Engineering

<p align="center">
  <img src="assets/demo.gif" alt="Autonomous Vehicle Simulation Demo" width="100%" />
</p>

---

## 🧠 What Is This Project About?

In this project you will build the control system for a self-driving car in a ROS 2 simulation. The car drives around a racetrack and your job is to make it stay on the path, control its speed, and complete laps as fast and accurately as possible.

You will work through a series of milestones, each building on the last:

1. **Explore the system** — Learn what topics the car publishes and subscribes to.
2. **Bring the car to life** — Implement the physics equations that describe how the car moves.
3. **Drive it manually** — Build a keyboard teleoperation node to drive the car yourself.
4. **Add cruise control** — Implement a PID speed controller so the car holds a steady speed.
5. **Make it autonomous** — Implement three different steering controllers (Lateral PID, Pure Pursuit, and MPC) so the car drives itself around the track.
6. **Monitor performance** — Build a lap analyzer that logs lap times, tracking error, and shows live graphs and 3D overlays in RViz.
7. **Report your results** — Compare your controllers and document your findings.

The car model is realistic: it has velocity as a state (not a direct input), meaning it accelerates and decelerates due to drag and friction — just like a real vehicle.

---

## 🚀 Quickstart

### 1. Build the Workspace
```bash
# Source ROS 2 Humble
source /opt/ros/humble/setup.bash

# Install build, simulation, and controller dependencies
sudo apt update && sudo apt install -y python3-colcon-common-extensions \
  python3-numpy python3-scipy ros-humble-robot-state-publisher \
  ros-humble-rviz2 ros-humble-xacro

# Build the workspace (bicycle_sim, bicycle_control, track_environment)
cd /path/to/bicycle_gym-main
colcon build --symlink-install
source install/setup.bash
```

In every new terminal, source the ROS distribution and built workspace:

```bash
source /opt/ros/humble/setup.bash
cd /path/to/bicycle_gym-main
source install/setup.bash
```

### 2. Launch Modes

| Mode | Launch Command | Section |
|---|---|---|
| **Base Simulation (CLI Testing)** | `ros2 launch bicycle_sim bicycle_sim.launch.py` | Milestones 1 & 2 |
| **Interactive Keyboard Teleop** | `ros2 launch bicycle_sim bicycle_sim.launch.py controller:=teleop` | Milestones 3 & 4 |
| **Lateral PID (Reactive)** | `ros2 launch bicycle_sim bicycle_sim.launch.py controller:=lateral_pid` | Milestone 5.2 |
| **Pure Pursuit (Geometric Preview)**| `ros2 launch bicycle_sim bicycle_sim.launch.py controller:=pure_pursuit` | Milestone 5.3 |
| **Extended Kinematic MPC (Optimal Preview)** | `ros2 launch bicycle_sim bicycle_sim.launch.py controller:=mpc` | Milestone 5.4 |

For keyboard teleoperation, start the keyboard driver in a second sourced terminal:

```bash
sudo apt install -y ros-humble-teleop-twist-keyboard
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

To request closed-loop cruise control in teleoperation mode:

```bash
ros2 launch bicycle_sim bicycle_sim.launch.py controller:=teleop use_cruise_control:=true
```

The launch file also supports `rviz:=false` and `analyzer:=false` to disable those nodes. For example:

```bash
ros2 launch bicycle_sim bicycle_sim.launch.py controller:=pure_pursuit rviz:=false
```

### 3. Inspecting the ROS Graph

Run these commands from a second sourced terminal while the simulation is running:

```bash
ros2 node list
ros2 topic list
ros2 topic info /state
ros2 topic info /throttle
ros2 topic info /steer
ros2 interface show nav_msgs/msg/Odometry
ros2 interface show std_msgs/msg/Float32
ros2 interface show geometry_msgs/msg/Twist
ros2 topic echo /state
```

### 4. Direct Actuator Testing

With the base simulation running, send actuator commands from another sourced terminal. Throttle/brake uses `[-1.0, 1.0]`; steering is in radians, with positive values turning left.

```bash
ros2 topic pub --once /throttle std_msgs/msg/Float32 "{data: 0.5}"
ros2 topic pub --once /steer std_msgs/msg/Float32 "{data: 0.30}"
ros2 topic pub --once /throttle std_msgs/msg/Float32 "{data: -1.0}"
```

### 5. Real-Time Telemetry & Graphing
```bash
# Install plotting and telemetry visualization tools
sudo apt update && sudo apt install -y ros-humble-plotjuggler-ros rqt-plot

# Inspect live signals in rqt_plot:
ros2 run rqt_plot rqt_plot /telemetry/cte /telemetry/speed

# Or launch PlotJuggler for multi-topic time-series analysis:
ros2 run plotjuggler plotjuggler
```

Some telemetry topics are only available after the corresponding analyzer work is complete.

---

## 🎯 Milestones at a Glance

- **Milestone 1**: Topic Discovery, Graph Inspection & Telemetry Plotting (`ros2 topic list / info`, `rqt_plot`, `plotjuggler`)
- **Milestone 2**: Extended Kinematic Bicycle Model & Euler Integration (`src/bicycle_sim/bicycle_sim/bicycle_model.py`)
- **Milestone 3**: Teleoperation Bridge & Open-Loop Driving (`src/bicycle_control/bicycle_control/teleop_bridge.py`)
- **Milestone 4**: Low-Level Powertrain Cruise Control (`src/bicycle_control/bicycle_control/longitudinal_pid.py`)
- **Milestone 5**: Autonomous Path Tracking — It's Time to Get the Car to Drive Autonomously!
  - **5.1**: High-Level Velocity Profiler & Path Curvature (`src/bicycle_control/bicycle_control/velocity_profiler.py`)
  - **5.2**: Steer Using Reactive Feedback (`src/bicycle_control/bicycle_control/lateral_pid.py`)
  - **5.3**: Steer Using Geometric Preview (`src/bicycle_control/bicycle_control/pure_pursuit.py`)
  - **5.4**: Steer Using Constrained Optimal Preview (Extended Kinematic MPC) (`src/bicycle_control/bicycle_control/mpc.py`)
  - **5.5**: Real-Time Telemetry, Graphing & RViz Dashboard Engineering (`src/track_environment/track_environment/lap_analyzer.py`)
- **Milestone 6**: Free Exploration & Reference Resources (Ackermann Kinematics, 3D Simulation, Nav2 MPPI)
- **Milestone 7**: Deliverable 1 — Repository Documentation (`README.md` Benchmark Report)
- **Milestone 8**: Deliverable 2 — Technical Video Walkthrough (3–5 Minute Demo)

---

## Milestone 6: Free Exploration & Reference Resources

To connect your work in this lab to industrial autonomous vehicle systems, modern simulators, and production ROS 2 frameworks, explore the following organized learning resources. These materials illustrate how the 2D planar kinematic bicycle model extends into multi-body dynamics, 3D physics engines, and advanced sampling-based predictive control.

---

### 1. Four-Wheel Ackermann Kinematics & `ros2_control`
*Explore multi-body steering geometry and industrial ROS 2 controller architectures.*

In a physical 4-wheel vehicle navigating a turn, the inside front wheel must turn sharper than the outside wheel because it follows a smaller turning radius ($R - W/2$ vs $R + W/2$). Forcing both wheels to the same angle causes tire scrub, tread wear, and energy loss.

$$\tan\delta_{inner} = \frac{L}{R - \frac{W}{2}}, \quad \tan\delta_{outer} = \frac{L}{R + \frac{W}{2}}$$

#### Curated Resources:
- [ROS 2 Control Mobile Robot Kinematics Guide](https://control.ros.org/humble/doc/ros2_controllers/doc/mobile_robot_kinematics.html) — Guide on modeling 4-wheel kinematics and visualizing full car models instead of simplified bicycle models.
- [ROS 2 Steering Controllers Library](https://control.ros.org/kilted/doc/ros2_controllers/steering_controllers_library/doc/userdoc.html) — Official documentation for Ackermann and bicycle steering controllers in `ros2_control`.
- [ros2_control_demos Example 11: Steered Wheel Base](https://control.ros.org/humble/doc/ros2_control_demos/example_11/doc/userdoc.html) — Industrial demonstration of steered-wheel bases and hardware interfaces.
- [ros2_control_demos Repository](https://github.com/ros-controls/ros2_control_demos) — Comprehensive reference suite for `ros2_control` implementations.
- [ROS 2 Controllers Official Repository](https://github.com/ros-controls/ros2_controllers/tree/master) — Upstream implementations of vehicle and chassis controllers.
- [Four-Wheel AMR Reference Implementation](https://github.com/abubakar-mughal97/four_wheel_amr) — 4-wheel mobile robot package with Ackermann steering.

---

### 2. Modern 3D Simulation Environments (Gazebo & MVSim)
*Bridge the gap between 2D planar kinematics and full 3D physics engines with tire friction dynamics.*

While kinematic models assume zero tire slip, physical vehicles experience tire deflection and friction saturation (Pacejka Magic Formula). 3D physics engines simulate suspension compliance, tire contact patches, sensor noise, and terrain.

#### Curated Resources:
- [Ackermann Vehicle in Modern Gazebo (Gz-Sim) & ROS 2](https://github.com/alitekes1/ackermann-vehicle-gzsim-ros2) ([Main Branch](https://github.com/alitekes1/ackermann-vehicle-gzsim-ros2/tree/main)) — Autonomous Ackermann vehicle simulation using modern Gazebo (Gz-Sim / Ignition) and ROS 2.
- [Classic Gazebo Ackermann Simulation](https://github.com/lucasmazzetto/gazebo_ackermann_steering_vehicle) — Classic Gazebo simulation showcasing physical Ackermann steering linkages.
- [Ackermann Autonomous Car Simulation](https://github.com/armando-genis/Ackermann-Autonomous-Car-Simulation) — Autonomous driving stack with Ackermann kinematics in simulation.
- [MVSim — Multi-Vehicle Simulator for ROS 2 Humble](https://docs.ros.org/en/humble/Tutorials/Advanced/Simulators/MVSim/Simulation-MVSim.html) — Lightweight, fast multi-vehicle dynamic simulator tailored for mobile robots and autonomous vehicles.

---

### 3. Stochastic Sampling-Based Predictive Control (Nav2 MPPI)
*Explore model predictive path integral control for non-linear vehicle systems.*

Model Predictive Path Integral (MPPI) control is an advanced algorithm that generates thousands of randomized candidate trajectories in parallel (using GPU or multi-core CPU)
#### Curated Resources:
- [Nav2 MPPI Controller](https://index.ros.org/p/nav2_mppi_controller/) — Production real-time MPPI controller package in the ROS 2 Navigation stack with dynamic obstacle avoidance and customizable cost functions.

---

### 💡 Synthesis Task for Your Report:
**In your `README.md` report, synthesize your takeaways from exploring these organized resources:**

1. **Kinematics vs Multi-Body**: 
   The 2D kinematic bicycle model simplifies the vehicle by representing the two front wheels and two rear wheels as single wheels on the centerline. In reality, a physical vehicle uses Ackermann kinematics where the inner wheel must turn at a sharper angle ($\delta_{inner}$) than the outer wheel ($\delta_{outer}$) to prevent tire scrubbing during cornering. `ros2_control` manages this complexity by utilizing hardware interfaces and specific steering controllers (like `ackermann_steering_controller`) that calculate and actuate the exact angle for each independent wheel joint, bridging the gap between simplified algorithms and physical hardware limits.

2. **2D vs 3D Simulation**: 
   2D kinematic simulations (like the one used in this project) are computationally lightweight and excellent for rapid prototyping of high-level control logic. However, they assume perfect traction (zero slip) and ignore vertical dynamics. Modern 3D physics engines like Gazebo or MVSim introduce multi-body dynamics, Pacejka tire friction models, suspension compliance, and sensor noise. While computationally expensive, 3D simulations are crucial for verifying that control algorithms (like MPC) won't cause the vehicle to spin out or roll over due to aggressive lateral acceleration before real-world deployment.

3. **Deterministic vs Sampling Control**: 
   Deterministic controllers like MPC use gradient-based optimization solvers (e.g., `scipy.optimize` used in this project) to find a single optimal trajectory. They are effective but can struggle with non-linear constraints or get trapped in local minima (resulting in bounding clipping warnings). On the other hand, sampling-based controllers like Nav2 MPPI generate thousands of random trajectory rollouts in parallel (leveraging GPUs). They evaluate the cost of each random path and average them. This makes MPPI much more robust to noise, highly flexible for dynamic obstacle avoidance, and free from the mathematical instability of complex optimization solvers.

---

## 🏆 Telemetry Benchmark Leaderboard

*(Telemetry Data collected via the custom `lap_analyzer` node in the ROS 2 simulation)*

| Controller Mode | Best Lap Time (s) | Top Speed (m/s) | Mean CTE (m) | Max CTE (m) | RMS CTE (m) | Laps Completed / Status |
|---|---|---|---|---|---|---|
| **Manual Teleoperation** | N/A | N/A | N/A | N/A | N/A | Testing Only |
| **Lateral PID (Reactive)** | 229.10 | 5.02 | 93.742 | 238.364 | 121.677 | 1 Lap Completed (Conservative) |
| **Pure Pursuit (Preview)** | 198.20 | 23.72 | 0.694 | 6.217 | 1.616 | 1 Lap Completed (**Best Overall**) |
| **Extended Kinematic MPC (Optimal)** | 226.88 | 23.00 | 104.027 | 198.372 | 140.507 | 1 Lap Completed (Solver limited) |

### 🔍 Controller Comparison & Observations:
* **Pure Pursuit** achieved the most robust performance, yielding the fastest lap time (**198.20s**) and the tightest path tracking (**0.694m Mean CTE**). Its geometric lookahead logic naturally scales with speed, making it highly effective.
* **Lateral PID** struggled significantly without preview information. It required extremely conservative speeds (max 5.02 m/s) to maintain stability, resulting in wide cornering and high CTE errors.
* **MPC** showed excellent straight-line acceleration (23.00 m/s) but suffered during sharp curves due to the computational overhead of the `scipy` optimization solver, which frequently hit boundary limits. With a faster real-time solver (e.g., C++ CasADi or OSQP), its performance would surpass Pure Pursuit.