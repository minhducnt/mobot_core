# mobot_core

**A research documentation project about smartphone-powered robotics, inspired by OpenBot, where a smartphone acts as the robot's brain for sensing, computation, communication, and control.**

---

## Project Description

`mobot_core` is a research and documentation repository exploring the concept of turning smartphones into robots. The project is inspired by the paper **"OpenBot: Turning Smartphones into Robots"** by Matthias Müller and Vladlen Koltun (2020). Rather than building expensive, purpose-built robots, OpenBot proposes reusing commodity smartphones — which contain high-quality cameras, powerful processors, GPS, IMU, and wireless connectivity — as the central computing and sensing platform for low-cost robotic systems.

This repository serves as the research foundation and technical documentation for a future implementation of `mobot_core`.

---

## Project Objectives

- Understand and document how smartphones can serve as the brain of a robot.
- Summarize and analyse the OpenBot research paper.
- Design a reference system architecture for smartphone-based robotics.
- Describe hardware and software requirements for a low-cost robot body.
- Identify practical use cases, limitations, and future research directions.
- Prepare a structured knowledge base for future implementation work.

---

## Why Smartphones for Robotics?

Modern smartphones offer a remarkable combination of capabilities that make them ideal as a robot brain:

| Capability | Benefit for Robotics |
|---|---|
| High-resolution cameras | Visual perception and object detection |
| IMU (accelerometer, gyroscope) | Motion sensing and stabilisation |
| GPS | Outdoor localisation and navigation |
| Powerful SoC (CPU + GPU + NPU) | On-device AI inference |
| Wi-Fi and Bluetooth | Communication with peripherals and remote systems |
| Large battery | Extended operation time |
| Mature OS (Android/iOS) | Rich software ecosystem |
| Low cost (commodity hardware) | Makes robotics accessible to researchers and students |

A mid-range smartphone costing under $100 offers more compute power than dedicated embedded boards costing several times as much, while also integrating sensors and wireless communication in a single package.

---

## High-Level Architecture

```
+-------------------------------+
|        Smartphone             |
|  +-------------------------+  |
|  |  Android Application    |  |
|  |  - Camera / Sensors     |  |
|  |  - AI/Perception Model  |  |
|  |  - Control Logic        |  |
|  +------------|------------+  |
|               | USB / BT      |
+---------------|---------------+
                |
+---------------|---------------+
|     Microcontroller (Arduino) |
|  - Motor Driver               |
|  - Speed Sensors              |
|  - Sonar / Ultrasonic         |
+-------------------------------+
                |
     Left Motor   Right Motor
```

The smartphone communicates with a low-cost microcontroller (e.g., Arduino Nano) over USB serial or Bluetooth. The microcontroller handles real-time motor control, speed sensing, and obstacle detection, while the smartphone handles high-level perception, planning, and AI inference.

---

## Documentation Structure

```
mobot_core/
├── README.md                     ← This file
├── .gitignore
├── docs/
│   ├── 01_introduction.md        ← Motivation and background
│   ├── 02_paper_summary.md       ← OpenBot paper summary
│   ├── 03_system_architecture.md ← System architecture
│   ├── 04_hardware_design.md     ← Hardware components
│   ├── 05_software_stack.md      ← Software layers
│   ├── 06_use_cases.md           ← Application use cases
│   ├── 07_limitations.md         ← Known limitations
│   └── 08_future_work.md         ← Future research directions
├── latex/
│   ├── main.tex                  ← Full LaTeX technical report
│   └── references.bib            ← BibTeX references
└── assets/
    └── README.md                 ← Diagrams and images placeholder
```

---

## Quick Start (Reading the Docs)

1. Start with [`docs/01_introduction.md`](docs/01_introduction.md) for motivation and background.
2. Read [`docs/02_paper_summary.md`](docs/02_paper_summary.md) for an overview of the OpenBot paper.
3. Explore [`docs/03_system_architecture.md`](docs/03_system_architecture.md) for the system design.
4. Review hardware and software details in `docs/04_hardware_design.md` and `docs/05_software_stack.md`.
5. See future plans in [`docs/08_future_work.md`](docs/08_future_work.md).
6. For a compiled report, compile `latex/main.tex` with pdflatex.

---

## References

- Müller, M., & Koltun, V. (2021). **OpenBot: Turning Smartphones into Robots**. In *Proceedings of the Conference on Robot Learning (CoRL)*. [arXiv:2008.10631](https://arxiv.org/abs/2008.10631)
- OpenBot GitHub repository: [https://github.com/isl-org/OpenBot](https://github.com/isl-org/OpenBot)
- Android ML Kit: [https://developers.google.com/ml-kit](https://developers.google.com/ml-kit)
- TensorFlow Lite: [https://www.tensorflow.org/lite](https://www.tensorflow.org/lite)

---

## License

This repository contains research documentation only. No production code is included. All referenced works belong to their respective authors.
