# Limitations

This document provides an honest assessment of the limitations and trade-offs of the `mobot_core` platform. Understanding these constraints is essential for setting realistic expectations and making informed decisions about where this platform is and is not appropriate.

---

## 7.1 Smartphone Compatibility

### Issue
Not all Android smartphones are equally capable. The system's performance depends heavily on the specific device used.

### Details
- **Processing power**: Older or budget phones may lack a GPU delegate or NPU, forcing AI inference onto the CPU, which significantly reduces frame rate (from ~30 fps to ~5 fps).
- **Camera quality**: Wide-angle lenses on some phones introduce significant distortion that degrades object detection accuracy.
- **USB OTG support**: Some devices do not support USB OTG (Host mode), making serial communication with the Arduino impossible without a workaround.
- **Android version**: Some APIs (NNAPI, CameraX features) require Android 8.0 or later.
- **RAM**: Phones with less than 3 GB RAM may struggle to run the app alongside background processes.

### Mitigation
- Publish a tested hardware compatibility list.
- Support Bluetooth as a fallback for devices without USB OTG.
- Provide CPU-only model variants optimised for lower-end devices.

---

## 7.2 Battery Life

### Issue
Operating both the robot motors and the smartphone simultaneously places a significant power demand on two separate battery systems.

### Details
- **Motor battery (LiPo)**: A 1000 mAh 2S LiPo lasts approximately 30–60 minutes under typical indoor driving conditions, less if high-speed manoeuvres are frequent.
- **Smartphone battery**: Running the camera, AI inference, and display continuously drains the phone battery quickly (~2–4 hours at 30% brightness with GPU inference active).
- **Charging overhead**: Maintaining the phone via a USB power bank adds weight and complexity.
- **No unified power management**: The two power systems are independent and may run out at different times.

### Mitigation
- Use a higher-capacity LiPo (2000–3000 mAh).
- Implement battery-aware control that reduces speed or inference frequency when battery is low.
- Explore unified power solutions (single LiPo powering everything via a 5V regulator).

---

## 7.3 Sensor Accuracy

### Issue
The sensors available on the smartphone and robot body are consumer-grade and have inherent inaccuracies.

### Details
- **IMU drift**: Smartphone gyroscopes and accelerometers suffer from bias drift over time. Without external correction (GPS, visual odometry), heading estimates degrade within minutes.
- **GPS**: Unavailable indoors; outdoor accuracy is ~3–5 m under open sky — insufficient for precise navigation.
- **Sonar (HC-SR04)**: Limited to single-direction obstacle detection with a relatively wide beam angle (~15°), leading to missed obstacles at oblique angles. Maximum reliable range is ~200 cm indoors.
- **Wheel encoders**: Subject to slip on smooth surfaces or sharp turns, reducing odometry accuracy.
- **Camera**: Poor performance in low-light conditions. No depth information without a dedicated depth sensor.

### Mitigation
- Fuse multiple sensor sources for more robust state estimation.
- Use visual odometry to correct IMU drift.
- Add an additional sonar or IR sensor for side obstacle detection.

---

## 7.4 Real-Time Performance

### Issue
AI inference and control loop latency may be insufficient for fast-moving scenarios.

### Details
- **Object detection latency**: On mid-range phones, MobileNet-SSD runs at ~50–100 ms per frame, limiting the effective control rate to 10–20 Hz.
- **Communication latency**: USB serial adds ~10–20 ms per command; Bluetooth adds ~20–50 ms.
- **End-to-end latency**: The total latency from camera capture to motor command reaching the wheels is typically 100–200 ms, which limits the maximum controllable speed.
- **Jitter**: Android is not a real-time operating system. Garbage collection, background processes, and OS scheduling can introduce unpredictable latency spikes.

### Mitigation
- Use the NNAPI or GPU delegate for faster inference.
- Reduce model input resolution to improve frame rate.
- Implement predictive control to compensate for latency.
- Use a dedicated real-time microcontroller (already in the design) for time-critical low-level control.

---

## 7.5 Mechanical Stability

### Issue
The 3D-printed chassis and consumer-grade motors introduce mechanical variability and fragility.

### Details
- **Motor matching**: TT gear motors are not precisely matched; left and right motors may have slightly different no-load speeds, causing drift when both are commanded equally.
- **Chassis flex**: PLA plastic chassis can flex under stress, affecting the accuracy of odometry and the alignment of sensors.
- **Phone mounting security**: Vibration can cause the phone to shift in its mount, affecting camera orientation.
- **Terrain limitations**: The differential drive chassis with small wheels struggles on carpet, uneven terrain, or thresholds.
- **Structural fragility**: PLA chassis can crack if the robot falls off a table or collides with a hard surface at speed.

### Mitigation
- Calibrate motor speeds using closed-loop PID control with wheel encoders.
- Use PETG or ABS for greater durability in high-stress prints.
- Add vibration-damping foam between the phone and its mount.

---

## 7.6 Safety and Reliability

### Issue
The system lacks robust safety mechanisms, which limits deployment in settings with people or valuable equipment nearby.

### Details
- **No emergency stop hardware**: There is no physical kill switch; the robot can only be stopped via the app or by removing the battery.
- **Bluetooth/Wi-Fi interference**: Wireless control can be disrupted by interference, potentially causing loss of control.
- **Crash handling**: If the Android app crashes, the Arduino continues executing the last received command until a new one arrives or the connection times out.
- **Failsafe**: The Arduino firmware should implement a watchdog timer that stops the motors if no command is received within a timeout period — this is not always implemented in simple versions.

### Mitigation
- Implement a hardware emergency stop button connected to the Arduino.
- Add a communication watchdog timeout in the Arduino firmware (motors stop if no command received within 500 ms).
- Test crash handling scenarios before deploying near people.

---

## 7.7 Hardware Variability

### Issue
The use of off-the-shelf, low-cost components introduces variability between builds.

### Details
- **Component quality variation**: Cheap TT motors, L298N clones, and HC-SR04 sensors from different suppliers can have different performance characteristics.
- **3D print variability**: Tolerances in 3D-printed parts affect motor mount alignment and fit.
- **Clone microcontrollers**: Arduino Nano clones may use different USB-to-serial chips (CH340 vs FTDI), requiring different drivers on the host computer.
- **Battery quality**: Cheap LiPo batteries may have significantly less capacity than rated, or may degrade quickly with repeated deep discharge cycles.

### Mitigation
- Document tested component sources and part numbers.
- Provide tolerance notes in 3D print files.
- Test all component variants before recommending them in the official BOM.

---

## 7.8 Summary

| Limitation | Severity | Impact | Mitigation Difficulty |
|---|---|---|---|
| Smartphone compatibility | Medium | Variable performance | Medium |
| Battery life | Medium | Limited run time | Low |
| Sensor accuracy | Medium | Noisy state estimates | High |
| Real-time performance | Medium | Speed/response limits | Medium |
| Mechanical stability | Low–Medium | Drift, fragility | Low |
| Safety and reliability | High | Deployment risk | Medium |
| Hardware variability | Low–Medium | Inconsistent builds | Low |

These limitations are inherent to the low-cost, commodity-hardware approach and should be understood as trade-offs rather than failures. For the primary use cases of education, research prototyping, and low-speed autonomous tasks, the platform remains highly capable and practical.
