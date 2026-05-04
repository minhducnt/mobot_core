# Software Stack

This document describes the software architecture of `mobot_core`, focusing on the Android application layer that runs on the smartphone, as well as the firmware on the Arduino microcontroller.

---

## 5.1 Overview

The software stack is split across two devices:

| Device | Language | Role |
|---|---|---|
| Smartphone (Android) | Kotlin / Java | High-level intelligence, perception, control |
| Arduino Nano | C++ (Arduino) | Low-level motor control, sensor reading |

The Android application is the primary focus of the software stack. It brings together camera input, AI inference, sensor data processing, and serial communication into a unified control loop.

---

## 5.2 Android Application Architecture

The Android application follows a layered architecture:

```
+---------------------------------------------+
|              User Interface Layer            |
|  - Camera preview                            |
|  - Control mode selection (Manual/Auto)      |
|  - Sensor data dashboard                     |
|  - Logging controls                          |
+---------------------------------------------+
|              Application Logic Layer         |
|  - Drive mode state machine                  |
|  - Control policy execution                  |
|  - Data logging manager                      |
+---------------------------------------------+
|              Perception Layer                |
|  - TensorFlow Lite inference engine          |
|  - Person detector (MobileNet-SSD)           |
|  - Path / navigation model                   |
+---------------------------------------------+
|              Sensor Collection Layer         |
|  - CameraX API (video capture)               |
|  - SensorManager (IMU, GPS)                  |
|  - USB serial reader (Arduino data)          |
+---------------------------------------------+
|              Communication Layer             |
|  - USB OTG serial driver                     |
|  - Bluetooth serial (optional)               |
|  - Wi-Fi (remote logging / model sync)       |
+---------------------------------------------+
```

---

## 5.3 Sensor Data Collection

### 5.3.1 Smartphone Sensors
The Android application subscribes to built-in sensors via the `SensorManager` API:

| Sensor | Android API | Data |
|---|---|---|
| Accelerometer | `TYPE_ACCELEROMETER` | 3-axis linear acceleration (m/s²) |
| Gyroscope | `TYPE_GYROSCOPE` | 3-axis angular velocity (rad/s) |
| Magnetometer | `TYPE_MAGNETIC_FIELD` | 3-axis magnetic field (μT) |
| Rotation Vector | `TYPE_ROTATION_VECTOR` | Device orientation quaternion |
| GPS | `LocationManager` | Latitude, longitude, altitude |

Sensor readings are timestamped and can be logged to a CSV file for post-processing or used directly by the control policy.

### 5.3.2 Robot Body Sensors
Data from the Arduino is received over USB serial and parsed by the app:

```
Incoming format: {l:<int>,r:<int>,s:<int>}
  l = left wheel speed (pulses/s)
  r = right wheel speed (pulses/s)
  s = sonar distance (cm)
```

---

## 5.4 Camera Input

The app uses **CameraX** (Jetpack library) for camera access, providing:

- A real-time preview surface for the UI.
- An `ImageAnalysis` use case that delivers frames to the AI inference pipeline at a configurable frame rate.
- Support for multiple cameras (front/back switching).

Frames are delivered as `ImageProxy` objects, which can be converted to `Bitmap` or passed directly to TensorFlow Lite as `ByteBuffer` input.

**Typical camera settings**:
- Resolution: 640×480 or 1280×720 (depending on model input requirements)
- Frame rate: 15–30 fps
- Format: YUV_420_888 (converted to RGB for model input)

---

## 5.5 AI / Perception Models

The app uses **TensorFlow Lite** for on-device inference. Models are stored as `.tflite` files in the app's `assets/` directory and loaded at runtime.

### 5.5.1 Person Detection Model
- **Architecture**: MobileNet-SSD V2
- **Input**: 300×300 RGB image
- **Output**: Bounding boxes, class labels, confidence scores
- **Inference time**: ~30–80 ms on a mid-range phone (depending on delegate)
- **Delegates**: CPU, GPU (via TFLite GPU delegate), or NNAPI (Android Neural Networks API)

### 5.5.2 Autonomous Navigation Model
- **Architecture**: Convolutional Neural Network (CNN)
- **Input**: Cropped forward-facing camera image
- **Output**: Left and right wheel speed commands (continuous values)
- **Training**: Imitation learning from human demonstrations
- **Inference time**: ~10–30 ms

New models can be added by placing a `.tflite` file in `assets/` and registering it in the model manager configuration.

---

## 5.6 Control Logic

The control logic determines what motor commands to send based on the current drive mode and perception outputs.

### 5.6.1 Drive Modes

| Mode | Description |
|---|---|
| **Manual** | User controls the robot via on-screen joystick or Bluetooth gamepad |
| **Person Following** | AI detects a person and steers towards them |
| **Autonomous Navigation** | CNN maps camera frames directly to wheel speeds |
| **Data Collection** | Human drives the robot while the app records camera + sensor data |

### 5.6.2 Person-Following Control Law

When a person is detected:
1. Compute the horizontal offset of the bounding box centre from the image centre.
2. Scale the offset to a steering correction value in the range [-1, 1].
3. Apply the correction to the base speed:
   ```
   left_speed  = base_speed * (1 + steering)
   right_speed = base_speed * (1 - steering)
   ```
4. Send the computed speeds to the Arduino.

If no person is detected for N consecutive frames, the robot stops or enters a search behaviour.

---

## 5.7 Communication with Robot Hardware

The app uses **USB serial** via the `usb-serial-for-android` library (or similar) to communicate with the Arduino Nano.

### Sending Commands
Motor commands are formatted and sent at each control loop iteration:

```kotlin
val command = "{l:${leftSpeed},r:${rightSpeed}}\n"
serialPort.write(command.toByteArray(), TIMEOUT_MS)
```

### Receiving Sensor Data
Incoming data from the Arduino is read on a background thread:

```kotlin
val buffer = ByteArray(64)
val len = serialPort.read(buffer, TIMEOUT_MS)
val message = String(buffer, 0, len)
parseSensorData(message)  // extracts l, r, s fields
```

The communication runs at **115200 baud** with a control loop period of ~50–100 ms.

---

## 5.8 Navigation and Workloads

### Person Following Workload
- Frame capture → person detection (MobileNet-SSD) → steering control → serial command
- Loop rate: ~10–15 Hz (limited by detection speed)
- CPU + GPU delegate usage: moderate

### Autonomous Navigation Workload
- Frame capture → image crop/resize → CNN inference → speed commands → serial command
- Loop rate: ~20–30 Hz (faster, lighter model)
- CPU or GPU delegate: low-moderate

### Data Collection Workload
- All sensors + camera logged to internal storage
- Human drives with gamepad; logs are labelled automatically
- Storage format: CSV for sensors, JPEG/PNG for images or video

---

## 5.9 Arduino Firmware Summary

The Arduino firmware (`firmware.ino`) handles:

1. **Serial command parsing**: Reads incoming `{l:XX,r:XX}` messages from the phone.
2. **Motor PWM output**: Maps speed values (0–255) to PWM duty cycle for each motor via the L298N.
3. **Direction control**: Sets IN1/IN2 pins on the L298N for forward/backward per motor.
4. **Encoder interrupts**: Counts encoder pulses using `attachInterrupt()` to measure speed.
5. **Sonar reading**: Triggers HC-SR04 and measures echo duration with `pulseIn()`.
6. **Sensor broadcast**: Sends `{l:XX,r:XX,s:XX}` messages back to the phone at ~10 Hz.

The firmware is intentionally kept minimal to ensure real-time reliability and easy modification.

---

## 5.10 Data Flow Summary

```
[Camera] ──→ [CameraX] ──→ [TFLite Model] ──→ [Control Policy]
                                                      |
[IMU/GPS] ─→ [SensorManager] ──────────────→ [State Estimator]
                                                      |
[Arduino] ─→ [USB Serial Reader] ──────────→ [Odometry / Obstacle]
                                                      |
                                              [Command Builder]
                                                      |
                                              [USB Serial Writer] ──→ [Arduino]
```

Each layer processes data independently and updates a shared robot state object, which the control policy reads at each iteration.
