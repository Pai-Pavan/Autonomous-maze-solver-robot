# Autonomous Maze Solver Robot

## Overview
This project implements a compact autonomous maze-solving robot using a **left-wall-following algorithm** for navigation in unknown environments. The system is designed with a **minimal-sensor approach**, relying on efficient logic and sensor fusion instead of hardware redundancy.

The robot is capable of making real-time navigation decisions, correcting drift, and performing precise turns using inertial feedback.

**Key Achievement:**  
Achieved reliable navigation in unknown maze environments without external localization or mapping.

---

## Hardware Used
- **Microcontroller:** Arduino (ATmega328P)
- **Motor Driver:** L298N
- **Sensors:**
  - Ultrasonic Sensors (Front + Left)
  - IR Sensors (for drift correction, angled at 45°)
  - MPU6050 (Gyroscope + Accelerometer)
- **Power Supply:** 3-cell Li-ion battery pack
- **Drive Mechanism:** Differential drive (2 DC motors + caster wheel)

---

## System Design

### Navigation Logic
- Implements **left-wall-following algorithm**
- Decision priority:
  1. Turn Left (if path exists)
  2. Move Forward
  3. Turn Right / U-turn (logical inference)

### Sensor Strategy
- Minimal sensor configuration (no right-side ultrasonic)
- Ultrasonic sensors used for path detection
- IR sensors used for real-time drift correction
- MPU6050 used for accurate angular rotation

### Turning Mechanism
- Gyro-based closed-loop turning (not delay-based)
- Bias calibration performed before every rotation
- Real-time yaw integration ensures ~90° accuracy

---

## Features
- Real-time decision making
- Sensor fusion (Ultrasonic + IR + Gyro)
- Drift correction during motion
- Accurate 90° turns using MPU6050
- Optimized hardware usage through algorithm design

---

## Code Structure
- `setup()` → Hardware initialization
- `loop()` → Main navigation logic
- `motorControl()` → Motor actuation
- `readUltrasonic()` → Distance measurement
- `rotateBot()` → Gyro-based turning

---

## How to Run
1. Upload the code to Arduino (ATmega328P)
2. Power the robot using Li-ion battery pack
3. Place the robot in a maze environment
4. Allow initialization (10s delay)
5. Robot starts autonomous navigation

---

## Future Improvements
- Implement maze mapping (DFS / Flood Fill)
- Add encoder-based odometry
- Replace L298N with efficient motor driver
- Improve sensor filtering and noise handling

---

## Author
Pavan Pai  
B.E Electronics and Communication Engineering  
BMS College of Engineering
