# Introduction

## 1.1 Motivation: The Case for Low-Cost Robotics

Robotics research has made remarkable advances over the past decade. However, the tools and hardware required to participate in cutting-edge robotics remain expensive and inaccessible to many researchers, educators, and developers — particularly in low-income regions or under-resourced institutions.

A standard research robot such as the TurtleBot 3, Boston Dynamics Spot, or a Universal Robots arm can cost anywhere from **$500 to hundreds of thousands of dollars**. This cost excludes the sensing suite (cameras, LiDAR, IMUs), the computing unit (Jetson, NUC, GPU workstation), and the software infrastructure required to run navigation, perception, and control algorithms.

This financial barrier limits who can participate in robotics research and development, and slows the democratisation of this transformative technology.

---

## 1.2 The Problem with Traditional Robot Designs

Traditional robot designs often follow a modular hardware architecture where each component is purchased or designed separately:

- **Compute unit**: A dedicated embedded SBC (Raspberry Pi, Jetson Nano) or workstation
- **Camera**: A dedicated USB or CSI camera module
- **Sensors**: Individual IMU, GPS, ultrasonic, or LiDAR modules
- **Communication**: Dedicated Wi-Fi adapter, ZigBee, or cellular module
- **Battery management**: Separate power board

The result is a system that is assembled from many parts, requires significant engineering effort to integrate, and adds considerable cost — even before considering the robot body itself.

Furthermore, these systems often require specialised software configurations and offer limited community support, making them less approachable for newcomers and students.

---

## 1.3 The Smartphone-as-Brain Concept

The core insight of the OpenBot project — and the foundation of `mobot_core` — is that **modern smartphones are already remarkable sensing and computing platforms** that can serve as a robot brain.

A typical mid-range Android smartphone (circa 2020–2025) contains:

- **Camera(s)**: Wide-angle, telephoto, and depth-sensing lenses, often with 12–108 MP resolution
- **IMU**: 3-axis accelerometer and gyroscope for motion sensing
- **GPS**: Accurate outdoor localisation
- **Processor**: Octa-core SoC with a dedicated Neural Processing Unit (NPU) capable of real-time AI inference
- **Connectivity**: Wi-Fi, Bluetooth, USB OTG, 4G/5G
- **Display**: Touchscreen for UI and debugging
- **Battery**: 3000–6000 mAh, sufficient for extended robot operation
- **Operating System**: Android, with access to a rich ecosystem of AI and robotics libraries

Rather than building or buying expensive purpose-built compute and sensor hardware, OpenBot proposes **mounting a smartphone on a simple low-cost robot body** and using the smartphone to handle all high-level intelligence — perception, planning, and communication — while a cheap microcontroller handles low-level motor control.

---

## 1.4 The `mobot_core` Project

`mobot_core` is a research and documentation project that explores this concept in depth. It aims to:

1. **Document the OpenBot concept** clearly and accessibly.
2. **Describe the architecture** of a smartphone-powered robot.
3. **Evaluate the feasibility** of this approach for low-cost robotics.
4. **Plan a future implementation** of a smartphone-based robot platform.

The goal is not to replace existing robot platforms, but to **lower the barrier to entry** for students, educators, and researchers who want to work with real robots without a large budget.

---

## 1.5 Scope of This Documentation

This documentation covers the following topics:

| Section | Topic |
|---|---|
| `02_paper_summary.md` | Summary of the OpenBot research paper |
| `03_system_architecture.md` | High-level system architecture |
| `04_hardware_design.md` | Physical robot body and components |
| `05_software_stack.md` | Software layers and application design |
| `06_use_cases.md` | Practical use cases and scenarios |
| `07_limitations.md` | Known limitations and trade-offs |
| `08_future_work.md` | Future implementation plans |

---

## References

- Müller, M., & Koltun, V. (2021). OpenBot: Turning Smartphones into Robots. *Conference on Robot Learning (CoRL)*. [arXiv:2008.10631](https://arxiv.org/abs/2008.10631)
