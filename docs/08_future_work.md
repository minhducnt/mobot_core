# Future Work

This document outlines the planned future development directions for `mobot_core`. The project is currently in its research and documentation phase. The following sections describe the concrete implementation steps and research directions that will form the next stages of the project.

---

## 8.1 Android Prototype Application

### Objective
Build a functional Android application that connects to the robot hardware and provides manual teleoperation as a starting point.

### Planned Features
- [ ] USB serial communication with Arduino Nano
- [ ] On-screen joystick for manual control
- [ ] Camera preview display
- [ ] Real-time sensor data dashboard (wheel speeds, sonar, IMU)
- [ ] Drive mode switcher (Manual / Auto)
- [ ] Logging toggle (start/stop data collection)

### Technical Stack
- **Language**: Kotlin
- **Camera**: CameraX (Jetpack)
- **Serial**: `usb-serial-for-android` library
- **UI**: Jetpack Compose or XML layouts
- **Target SDK**: Android 8.0+ (API 26+)

### Milestones
1. Basic serial communication and motor command sending.
2. Camera preview with frame capture.
3. On-screen joystick and manual drive.
4. Sensor data overlay and logging.

---

## 8.2 Robot Control Module

### Objective
Develop and publish the Arduino firmware that manages motor control, odometry, and sensor interfacing.

### Planned Features
- [ ] USB serial command parser (`{l:XX,r:XX}` format)
- [ ] PWM motor control via L298N
- [ ] Closed-loop PID speed controller using wheel encoders
- [ ] HC-SR04 sonar obstacle detection
- [ ] Watchdog timeout (stop motors if no command received within 500 ms)
- [ ] Sensor data broadcast to smartphone

### Design Goals
- Keep firmware under 300 lines of C++.
- Support both Arduino Nano and Arduino Uno.
- Document all serial message formats clearly.

### Testing Plan
- Unit test command parsing logic with a PC serial terminal.
- Verify PID controller by driving the robot at constant speed and measuring encoder counts.
- Verify sonar readings against a ruler.

---

## 8.3 Computer Vision Module

### Objective
Integrate AI/ML models into the Android app for real-time perception tasks.

### Phase 1: Person Detection
- [ ] Integrate TensorFlow Lite runtime.
- [ ] Add MobileNet-SSD V2 (COCO-pretrained) for person detection.
- [ ] Implement bounding box overlay on camera preview.
- [ ] Implement person-following control law based on bounding box position.

### Phase 2: Custom Model Training
- [ ] Collect a dataset using the robot's data collection mode.
- [ ] Train a navigation CNN (image → steering command) using Keras/PyTorch.
- [ ] Convert the trained model to TensorFlow Lite format.
- [ ] Integrate and test the model on the robot.

### Phase 3: Model Optimisation
- [ ] Enable GPU delegate for faster inference.
- [ ] Evaluate NNAPI delegate on supported devices.
- [ ] Benchmark inference latency on target devices.
- [ ] Explore quantisation (INT8) for further speed improvement.

---

## 8.4 SLAM and Navigation Support

### Objective
Enable the robot to build a map of its environment and navigate autonomously without pre-recorded demonstrations.

### Research Directions
- **Visual SLAM (vSLAM)**: Use only the smartphone camera to simultaneously localise and map the environment (e.g., ORB-SLAM3, RTAB-Map).
- **Lidar-free navigation**: Combine visual odometry, IMU data, and sonar for a map-less local obstacle avoidance system.
- **Graph-based navigation**: Build a topological map of waypoints and navigate between them.

### Challenges
- Limited computational resources for full SLAM on a smartphone.
- Depth estimation is difficult without a dedicated depth sensor.
- Android real-time guarantees are insufficient for tight SLAM loops.

### Potential Approaches
- Run a lightweight vSLAM algorithm (e.g., SuperPoint + lightweight graph matching) on the NPU.
- Offload SLAM to a companion compute device (e.g., Raspberry Pi Zero 2W) while keeping the smartphone for perception.
- Use ARCore (Google's built-in smartphone AR framework) as a ready-made visual-inertial odometry solution.

---

## 8.5 Dataset Collection Pipeline

### Objective
Build a structured pipeline for collecting, annotating, and sharing datasets from the robot.

### Planned Features
- [ ] Automatic timestamped recording of camera frames, IMU, GPS, and control inputs.
- [ ] Configurable recording intervals and resolution.
- [ ] On-device dataset review and annotation tool.
- [ ] Export to standard formats (rosbag, HDF5, CSV + image folders).
- [ ] Dataset sharing via cloud upload (Google Drive, S3).

### Research Value
A large, diverse dataset collected on real robot hardware is valuable for training and benchmarking imitation learning and navigation models. Making this dataset open and reproducible will benefit the broader research community.

---

## 8.6 Simulation Environment

### Objective
Create a simulation environment that mirrors the `mobot_core` hardware, enabling model training and testing without physical hardware.

### Options
- **Gazebo**: Mature ROS-based simulator with physics, sensors, and visualisation.
- **PyBullet**: Lightweight Python-based physics engine suitable for RL training.
- **CARLA**: Photo-realistic simulator primarily for autonomous vehicles, adaptable for small robots.
- **Isaac Sim (NVIDIA)**: High-fidelity simulator with GPU-accelerated physics, good for sim-to-real transfer.

### Planned Steps
- [ ] Model the OpenBot-compatible robot body in a simulator.
- [ ] Implement a simulated Arduino interface (serial communication emulation).
- [ ] Add a simulated smartphone camera view.
- [ ] Enable training of navigation models entirely in simulation.
- [ ] Evaluate sim-to-real transfer accuracy.

---

## 8.7 Hardware Testing Plan

### Objective
Systematically validate the hardware design before committing to a full software implementation.

### Test Plan

| Test | Method | Pass Criteria |
|---|---|---|
| Motor control | Send PWM commands, observe wheel spin | Both motors respond correctly |
| Serial communication | PC → Arduino serial terminal | Commands parsed, sensors reported |
| Encoder accuracy | Drive 1 m, count pulses vs. ruler | < 5% distance error |
| Sonar accuracy | Measure known distances (10–100 cm) | < 10% range error |
| USB OTG connection | Connect phone to Arduino, open app | App detects Arduino, connection established |
| Battery runtime | Drive continuously, measure time | > 30 min on 1000 mAh LiPo |
| Straight-line driving | Drive 2 m without correction | < 10 cm lateral deviation |
| Person following (indoor) | 10 trials, person moves 1 m away | > 70% success rate |

### Hardware Versions to Test
- Arduino Nano (genuine and CH340 clone)
- L298N motor driver (different suppliers)
- TT motors (yellow gear motor, various torque ratings)
- HC-SR04 sonar sensor

---

## 8.8 Summary Roadmap

| Phase | Goal | Status |
|---|---|---|
| Phase 0 | Research & documentation | ✅ In progress |
| Phase 1 | Arduino firmware & hardware assembly | 🔲 Planned |
| Phase 2 | Android app (manual control) | 🔲 Planned |
| Phase 3 | Computer vision (person following) | 🔲 Planned |
| Phase 4 | Dataset collection pipeline | 🔲 Planned |
| Phase 5 | Autonomous navigation model | 🔲 Planned |
| Phase 6 | SLAM / mapping support | 🔲 Planned |
| Phase 7 | Simulation environment | 🔲 Planned |
| Phase 8 | Extended evaluation & publication | 🔲 Planned |

Each phase builds upon the previous, allowing incremental validation of the system. The research and documentation produced in Phase 0 (this repository) will guide all subsequent implementation work.
