# Hardware Design

This document describes the physical hardware components of a `mobot_core`-compatible robot, based on the OpenBot design. The goal is to assemble a functional robot body for under **$50 USD** using readily available, off-the-shelf components and 3D-printed parts.

---

## 4.1 Design Philosophy

The hardware design follows three guiding principles:

1. **Low cost**: Every component should be affordable and available globally.
2. **Simplicity**: The assembly should be achievable by a hobbyist with basic tools.
3. **Reproducibility**: The design should use standard parts and openly shared 3D models.

The robot body itself does not need to be sophisticated — its role is purely physical actuation. All intelligence resides in the smartphone.

---

## 4.2 Robot Body Overview

The robot uses a **differential drive** configuration:

- Two powered wheels (left and right), each driven by a DC gear motor.
- One or two passive caster wheels at the front and/or rear for balance.
- The chassis is flat and rectangular, large enough to mount the electronics and the smartphone.

```
         [Smartphone mount]
              [ ][ ][ ]
   ___________________________________
  |  [Motor L]          [Motor R]    |
  |    [Encoder]      [Encoder]      |
  |                                  |
  |   [Arduino]   [Motor Driver]     |
  |   [Battery]                      |
  |__________________________________|
         [Caster wheel]
```

---

## 4.3 3D-Printed Chassis

The chassis is designed to be printed on a standard FDM 3D printer using PLA or PETG filament.

| Property | Specification |
|---|---|
| Material | PLA or PETG |
| Print time | ~4–8 hours (depending on printer) |
| Layer height | 0.2 mm recommended |
| Infill | 20–30% |
| Support | Minimal (flat design) |

The chassis includes:
- Motor mount slots for press-fit or screw-mounted gear motors.
- Mounting holes for the Arduino Nano and motor driver board.
- A raised platform or rail for the smartphone mount.
- Cable routing channels for clean wiring.

The OpenBot STL files are publicly available at [https://github.com/isl-org/OpenBot](https://github.com/isl-org/OpenBot).

---

## 4.4 Arduino / Microcontroller Role

The **Arduino Nano** (or compatible clone) is the low-level controller for the robot body. It is responsible for:

- Receiving speed commands from the smartphone over USB serial.
- Setting PWM output to the motor driver to control wheel speed and direction.
- Reading optical encoder pulses to measure wheel speed (odometry).
- Reading the ultrasonic distance sensor.
- Sending sensor feedback to the smartphone.

The Arduino firmware is simple and typically fewer than 200 lines of C++. It operates as a command executor: it parses incoming serial messages, updates motor outputs, and streams sensor readings.

---

## 4.5 Motors and Motor Driver

### DC Gear Motors
- **Type**: TT gear motors (also called "yellow motors")
- **Voltage**: 3V–6V (operated at ~6V from the battery)
- **Speed**: ~100–200 RPM (with gear reduction)
- **Quantity**: 2 (one per drive wheel)
- **Cost**: ~$2–4 per motor

### Motor Driver
- **Module**: L298N dual H-bridge motor driver
- **Input voltage**: Up to 46V (powered from robot battery)
- **Logic voltage**: 5V (from Arduino or USB power)
- **Control**: PWM speed + direction pin per channel
- **Cost**: ~$2–3

The L298N module can drive two DC motors independently, enabling differential steering.

---

## 4.6 Battery

A **2S LiPo battery (7.4V nominal)** is used to power the drive motors through the L298N motor driver. The motor driver's onboard 5V regulator powers the Arduino Nano.

The smartphone is powered separately via a **USB power bank** (5V/2A output), which can be taped or mounted to the chassis.

| Power Rail | Source | Powers |
|---|---|---|
| 7.4V | 2S LiPo | Motors (via L298N) |
| 5V | L298N 5V output | Arduino Nano |
| 5V | USB power bank | Smartphone charging |

**Estimated runtime**: 30–60 minutes depending on motor load and battery capacity (typically 1000–2000 mAh for LiPo).

---

## 4.7 Speed Sensors (Wheel Encoders)

Each drive wheel has an **optical encoder disc** attached to the motor shaft, paired with an infrared photointerrupter sensor (or slotted optocoupler).

- **Type**: Slotted disc encoder + IR photointerrupter (or magnetic hall-effect encoder)
- **Resolution**: Typically 20 slots per revolution
- **Output**: Digital pulse signal (read by Arduino interrupt pin)
- **Purpose**: Measure wheel rotation speed and distance for odometry

The Arduino firmware uses hardware interrupts to count encoder pulses, providing accurate wheel speed measurement without polling overhead.

---

## 4.8 Sonar / Ultrasonic Sensor

An **HC-SR04 ultrasonic distance sensor** is mounted at the front of the robot to detect obstacles.

| Property | Value |
|---|---|
| Model | HC-SR04 |
| Range | 2 cm – 400 cm |
| Accuracy | ±3 mm |
| Interface | Trigger + Echo digital pins |
| Cost | ~$1 |

The sensor triggers a sound pulse and measures the echo return time to compute distance. The Arduino reads this value and forwards it to the smartphone, which can use it to stop the robot before a collision.

---

## 4.9 Smartphone Mount

The smartphone is mounted vertically (portrait) or horizontally (landscape) at the front of the chassis, facing forward for camera-based perception.

The mount is a **3D-printed bracket** designed to:
- Hold the smartphone securely without blocking the camera lens.
- Allow the USB OTG cable to connect to the charging port.
- Be adjustable for different phone sizes.

Off-the-shelf phone holders (e.g., clamp-style holders used for bicycles) can also be repurposed.

---

## 4.10 Hardware Cost Summary

| Component | Estimated Cost (USD) |
|---|---|
| 3D-printed chassis | $2–5 (filament cost) |
| TT gear motors × 2 | $4–8 |
| L298N motor driver | $2–3 |
| Arduino Nano (or clone) | $3–5 |
| HC-SR04 sonar sensor | $1–2 |
| Wheel encoder discs × 2 | $1–2 |
| 2S LiPo battery (1000 mAh) | $8–12 |
| USB power bank (5000 mAh) | $8–12 |
| Wheels and casters | $3–5 |
| USB OTG cable | $1–2 |
| Miscellaneous (wires, screws) | $2–3 |
| **Total (without smartphone)** | **~$35–59** |

The smartphone (assumed to be already owned or purchased separately) brings the intelligence layer at no additional hardware cost to the robot body.

---

## 4.11 Hardware Variability and Alternatives

The design is intentionally flexible:

- The **Arduino Nano** can be replaced by an ESP32 (adds Wi-Fi/Bluetooth natively), a Raspberry Pi Pico, or any other microcontroller with a UART interface.
- The **L298N** can be replaced by a TB6612FNG or DRV8833 for a smaller form factor.
- The **LiPo battery** can be replaced by AA NiMH packs or 18650 Li-ion cells if LiPo handling is a concern.
- The **3D-printed chassis** can be replaced by laser-cut acrylic or wood for institutions without 3D printers.

This flexibility ensures that `mobot_core` hardware can be adapted to local availability and budget constraints.
