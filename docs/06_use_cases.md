# Use Cases

This document describes the practical applications and use cases for `mobot_core` — a smartphone-powered robotic platform. These use cases demonstrate the versatility of the system across education, research, and applied robotics contexts.

---

## 6.1 Overview

The low cost and accessibility of `mobot_core` make it suitable for a wide range of scenarios where traditional robots would be too expensive, complex, or fragile to deploy. The platform trades raw performance for affordability and ease of use, making it ideal for the following applications.

---

## 6.2 Education

### Target Audience
Students from secondary school through university level, coding bootcamp participants, and self-learners interested in robotics and AI.

### How `mobot_core` Supports Education

| Learning Goal | How the Platform Helps |
|---|---|
| Introduction to robotics | Build and control a real robot for under $50 |
| Programming (Kotlin/Java/C++) | Write Android app logic and Arduino firmware |
| AI and machine learning | Deploy TensorFlow Lite models to a real device |
| Sensor fusion | Work with camera, IMU, and ultrasonic data |
| Computer vision | Implement object detection and tracking |
| Embedded systems | Program microcontroller PWM, interrupts, UART |

### Example Exercises
- Build the robot body and flash the Arduino firmware.
- Write a simple gamepad teleoperation app.
- Train a colour-based line-following model and deploy it.
- Implement a simple PID controller for straight-line driving.

### Why It Works for Education
A $50 robot body + a student's own smartphone is far more accessible than traditional lab equipment. Students can keep the robot they build, encouraging continued learning at home.

---

## 6.3 Robotics Research

### Target Audience
Academic researchers, graduate students, and hobbyist researchers.

### Research Applications

| Research Area | Application |
|---|---|
| Imitation learning | Collect human demonstrations and train driving policies |
| Reinforcement learning | Use the robot as a physical test platform for RL agents |
| Visual odometry | Estimate motion from camera frames without GPS |
| Obstacle avoidance | Test learned or rule-based avoidance strategies |
| Multi-robot systems | Deploy multiple low-cost units for swarm experiments |
| Dataset collection | Collect large-scale real-world driving datasets cheaply |

### Advantages over Lab Robots
- **Replicability**: Multiple robots can be built for the cost of one traditional research robot.
- **Real-world grounding**: Physical robots expose models to real sensor noise and dynamics.
- **Open ecosystem**: Android and TFLite support a wide range of community models.

---

## 6.4 Person Following

### Description
The robot detects and follows a specific person in real time using the smartphone's camera and an on-device object detection model.

### How It Works
1. The robot is placed on the floor and the smartphone app is started.
2. A person stands in front of the robot.
3. The person detection model (MobileNet-SSD) identifies the person's bounding box.
4. The control law adjusts wheel speeds to centre the person in the camera frame.
5. The robot follows the person as they move.

### Applications
- **Care robotics**: Accompany elderly or mobility-impaired individuals.
- **Delivery assistance**: Follow a human through a facility carrying supplies.
- **Photography**: Auto-follow a subject for hands-free filming.
- **Research baseline**: Use as a benchmark for person-tracking algorithms.

### Performance (OpenBot reference)
- Indoor success rate: ~87%
- Outdoor success rate: ~78%
- Maximum following speed: ~0.5 m/s

---

## 6.5 Autonomous Navigation

### Description
The robot navigates along a path or through an environment without human input, guided only by its camera and a learned navigation model.

### How It Works
1. A human drives the robot along the desired path using a gamepad, while the app records camera frames and control inputs.
2. A CNN is trained on this demonstration data to map images to steering commands (imitation learning).
3. The trained model is deployed to the phone and the robot autonomously replicates the path.

### Applications
- **Indoor delivery**: Navigate fixed routes in offices or warehouses.
- **Security patrolling**: Autonomously traverse a predetermined patrol route.
- **Research testbed**: Evaluate imitation learning and generalisation.

### Extensions
- With additional training data, the model can generalise to new environments.
- SLAM-based navigation (future work) could replace learning-based path following for structured environments.

---

## 6.6 Low-Cost Robot Experimentation

### Description
`mobot_core` serves as a platform for experimenting with ideas that would be too costly or risky to try on expensive robots.

### Experiment Categories

| Experiment Type | Description |
|---|---|
| New control algorithms | Test novel PID, MPC, or RL controllers cheaply |
| Sensor modalities | Add and test new sensors (thermal camera, microphone array) |
| Communication protocols | Experiment with Bluetooth, Wi-Fi, or 5G teleoperation |
| Hardware modifications | Test new chassis designs, wheel configurations |
| Energy management | Explore dynamic speed scaling for battery efficiency |

### Advantages
- Failure is low cost: a crash costs cents in parts, not thousands of dollars.
- Rapid iteration: 3D-printed parts can be reprinted and revised quickly.
- Hackable: the Arduino firmware and Android app are easy to modify.

---

## 6.7 Assistive and Social Robotics

### Description
With appropriate software, `mobot_core` can serve as a simple social or assistive robot that interacts with users using the smartphone's screen, speaker, and microphone.

### Potential Features
- Display a face or emotion on the smartphone screen.
- Use text-to-speech for verbal communication.
- Use speech recognition for voice commands.
- Follow a person and deliver simple spoken responses.

### Applications
- Companion robot for children or elderly users.
- Interactive museum or event guide robot.
- Low-cost telepresence robot (navigate remotely, video call face-to-face).

---

## 6.8 Summary Table

| Use Case | Skill Level Required | Cost | Key Technology |
|---|---|---|---|
| Education | Beginner–Intermediate | Low (~$50) | Arduino, Android, TFLite |
| Robotics research | Intermediate–Advanced | Low | Imitation learning, CV |
| Person following | Intermediate | Low | Object detection, control |
| Autonomous navigation | Advanced | Low | CNN, imitation learning |
| Low-cost experimentation | Any | Very low | Modular hardware/software |
| Assistive robotics | Intermediate | Low | TTS, speech recognition |
