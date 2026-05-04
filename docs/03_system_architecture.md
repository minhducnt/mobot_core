# System Architecture

This document describes the overall system architecture of `mobot_core`, a smartphone-powered robotic platform. The architecture is inspired by the OpenBot design and separates concerns between a high-level intelligence layer (the smartphone) and a low-level actuation layer (the microcontroller).

---

## 3.1 Architectural Overview

The system can be conceptualised as two cooperating layers connected via a serial communication link:

```
+-------------------------------------------------------------+
|                        SMARTPHONE                           |
|                                                             |
|  +-----------------+   +-------------------+               |
|  |  Camera / IMU   |   |   Android App     |               |
|  |  GPS / Sensors  +-->|  - Perception     |               |
|  +-----------------+   |  - AI Inference   |               |
|                        |  - Control Logic  |               |
|                        |  - UI / Logging   |               |
|                        +--------+----------+               |
|                                 |                           |
+-------------------------------- | ---------------------------+
                                  |  USB Serial (OTG)
+-------------------------------- | ---------------------------+
|                        MICROCONTROLLER (Arduino Nano)       |
|                                 |                           |
|                        +--------+----------+               |
|                        |  Serial Parser    |               |
|                        |  Motor Commands   |               |
|                        |  Speed Sensors    |               |
|                        |  Sonar / Obstacle |               |
|                        +---+----------+----+               |
|                            |          |                     |
+--------------------------- | -------- | -------------------+
                             |          |
                    +--------+--+  +----+------+
                    | Left Motor|  |Right Motor|
                    +-----------+  +-----------+
```

---

## 3.2 Smartphone as the Brain

The smartphone is the most critical component in the architecture. It handles all cognitive tasks:

### 3.2.1 Perception
- Uses the smartphone's rear camera to capture real-time video frames.
- Runs AI models (TensorFlow Lite) directly on-device using the CPU, GPU, or Neural Processing Unit (NPU).
- Detects objects, people, obstacles, and path features.

### 3.2.2 State Estimation
- Reads data from built-in sensors: accelerometer, gyroscope, and magnetometer.
- Fuses sensor data for orientation estimation and dead-reckoning navigation.

### 3.2.3 Control Logic
- Interprets perception outputs to generate control signals.
- Computes left and right motor speed commands using control policies (rule-based or learned).
- Handles user input (manual driving via on-screen joystick or Bluetooth gamepad).

### 3.2.4 Communication
- Sends motor commands to the microcontroller over USB serial (USB OTG).
- Receives sensor feedback (wheel speed, sonar distance) from the microcontroller.
- Optionally communicates with a remote server over Wi-Fi for logging or model updates.

---

## 3.3 Robot Body as the Physical Platform

The robot body is a lightweight mechanical structure that:

- Provides mounting points for the smartphone, microcontroller, motors, and battery.
- Uses a **differential drive** configuration: two independently controlled drive wheels, plus one or two passive caster wheels.
- Is inexpensive and reproducible using **3D printing** or off-the-shelf parts.

The mechanical simplicity of the differential drive allows the control problem to be reduced to specifying two scalar values: left wheel speed and right wheel speed.

---

## 3.4 Camera and Sensors for Perception

| Sensor | Location | Role |
|---|---|---|
| Rear camera | Smartphone | Visual input for AI models |
| Accelerometer | Smartphone | Motion and tilt detection |
| Gyroscope | Smartphone | Angular velocity, orientation |
| Magnetometer | Smartphone | Compass heading |
| GPS | Smartphone | Outdoor localisation |
| Ultrasonic (HC-SR04) | Robot body (front) | Obstacle detection |
| Wheel encoders | Robot body | Odometry (speed and distance) |

The smartphone's built-in sensors provide rich state information with no additional hardware cost. The robot body sensors (sonar and encoders) provide low-level physical feedback that the smartphone cannot measure directly.

---

## 3.5 Mobile Processor for Onboard AI

Modern smartphones include powerful System-on-Chip (SoC) designs with multiple processing units:

- **CPU**: General-purpose computation, application logic.
- **GPU**: Parallel computation for image processing and neural network inference.
- **NPU / DSP**: Dedicated hardware accelerator for AI inference (e.g., Qualcomm Hexagon DSP, Google Tensor TPU).

TensorFlow Lite and ONNX Runtime can target these hardware accelerators via platform delegates, enabling real-time inference at 15–30+ fps for MobileNet-scale models.

This on-device inference capability eliminates the need for a network connection during operation and reduces latency compared to cloud-based inference.

---

## 3.6 Communication Between Smartphone and Microcontroller

The smartphone communicates with the Arduino Nano via **USB serial** using Android's USB Host API and USB OTG (On-The-Go) capability. The communication protocol is a simple text-based format:

**Commands from smartphone to Arduino** (example):
```
{l:150,r:150}   ← Set left and right motor speeds (0–255)
{l:0,r:0}       ← Stop
```

**Data from Arduino to smartphone** (example):
```
{l:148,r:152,s:45}   ← Actual left/right speed, sonar distance in cm
```

An alternative or complementary communication channel is **Bluetooth** (e.g., HC-05 or HC-06 module on the Arduino), which avoids the need for a USB cable but introduces slightly higher latency.

---

## 3.7 Motor Control and Navigation Flow

The end-to-end control flow is as follows:

```
1. Camera captures frame
         ↓
2. AI model processes frame (object detection / path detection)
         ↓
3. Control policy computes desired left/right speed
         ↓
4. Android app sends command to Arduino via USB serial
         ↓
5. Arduino sets PWM duty cycle on motor driver
         ↓
6. Motors turn at commanded speed
         ↓
7. Wheel encoders measure actual speed → feedback to Arduino
         ↓
8. Arduino reports sensor data back to smartphone
         ↓
9. Smartphone updates state estimate → repeat from step 1
```

This control loop runs at approximately 10–30 Hz, depending on AI inference speed and communication latency.

---

## 3.8 Summary

| Layer | Component | Responsibility |
|---|---|---|
| Intelligence | Smartphone | Perception, planning, AI, logging |
| Communication | USB / Bluetooth | Command and sensor data exchange |
| Control | Arduino Nano | Motor PWM, encoder reading, sonar |
| Actuation | DC Motors | Physical movement |
| Sensing (physical) | Encoders, Sonar | Low-level feedback |

This layered architecture makes the system modular and extensible. New AI models can be deployed to the smartphone without any hardware changes. The microcontroller firmware can be updated independently to support new sensors or actuators.
