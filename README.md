# 🚗 Kookmin Self Driving (Xycar) – Competition Code

<div align="center">
  <p>
    <img src="https://img.shields.io/badge/Platform-XYCAR-blue?logo=ros" alt="Platform" />
    <img src="https://img.shields.io/badge/Language-Python%20%7C%20ROS-00599C?logo=python" alt="Language" />
    <img src="https://img.shields.io/badge/API-OpenCV%20%7C%20LiDAR-red" alt="API" />
    <img src="https://img.shields.io/badge/Status-Completed-success" alt="Status" />
  </p>
</div>

---

## 📝 Project Overview
This repository contains autonomous driving software used in the **Kookmin University Self-Driving Competition**.

Main entrypoint is `driving.py`, which integrates:
- **Camera-based lane following** (HSV filtering + Bird's-Eye View + sliding window histogram + PD steering)
- **Traffic-light start trigger** (green light detection)
- **LiDAR-based rubber-cone/guide navigation** (pure pursuit style steering)
- **LiDAR-based vehicle detection & lane-change avoidance** (relative distance/velocity estimation + state machine flags)

---

## 🚀 Key Features

### 1) Lane Following (Camera)
Pipeline implemented in `driving.py`:
- **HSV lane color filtering** (yellow + white)
- **Perspective transform (BEV)** using `SOURCE_POINTS` -> `DESTINATION_POINTS`
- **Thresholding on HLS-L channel**
- **Sliding window histogram** to sample left/right lane positions (`nwindows`, `margin`, `minpix`)
- **PD-style steering** using:
  - `error`: center offset
  - `degree`: target heading angle derived from lane geometry
  - `angle = error * DX_GAIN + degree * DEGREE_GAIN`

### 2) Start Control (Traffic Light)
- Waits for green signal (via `sinhodeng.py`) before starting the main driving loop.

### 3) Rubber / Cone Navigation (LiDAR)
- `rubber.py` (`CurveNavigator`) tracks left/right guide structures in LiDAR sectors,
  clusters points, and drives using a pursuit-style steering approach.

### 4) Overtake / Avoid (LiDAR)
- `avoid_wd_lidar.py` estimates:
  - relative distance (`front_car_dis`)
  - relative velocity (`front_car_vel`)
- triggers lane-change avoidance using flags:
  - `lane_change_flag`, `start_avoid_flag`, `go_back_flag`

---

## 🧩 ROS Interfaces (driving.py)

### Subscribe
- `/usb_cam/image_raw/` (`sensor_msgs/Image`)
- `/scan` (`sensor_msgs/LaserScan`)

### Publish
- `xycar_motor` (`xycar_msgs/XycarMotor`)
  - `angle` (steering), `speed`

---

## 📂 Repository Structure
```text
.
├── driving.py           # Main integrated driving node (competition)
├── sinhodeng.py         # Traffic light (green) detection
├── rubber.py            # LiDAR guide/rubber-cone navigation + lane-change trigger
├── avoid_wd_lidar.py    # Relative speed/distance + avoidance control flags
├── hough_drive.py       # Alternative lane detection (Hough-based) (experimental)
├── hough.py             # Legacy/experimental lane driving
├── racing.py            # Alternative lane following (experimental)
├── test.py              # Test variant (experimental)
└── motortest.py         # Motor/vision experiments (not used in main)
```

---

## 🛠 Requirements
- **ROS1** environment (`rospy`)
- Python 3
- ROS messages:
  - `sensor_msgs`, `cv_bridge`, `xycar_msgs`
- Python libs:
  - `numpy`, `opencv-python`, `matplotlib`

---

## ▶️ How to Run (Example)
1) Start ROS core:
```bash
roscore
```

2) Make sure camera and LiDAR topics are published:
- Camera: `/usb_cam/image_raw/`
- LiDAR: `/scan`

3) Run:
```bash
python3 driving.py
```

---

## ⚙️ Tuning Points
These values are environment-dependent and usually need tuning per track/camera setup:
- Perspective transform: `SOURCE_POINTS`, `DESTINATION_POINTS`
- HSV thresholds: `LOW_YELLOW/HIGH_YELLOW`, `LOW_WHITE/HIGH_WHITE`
- Binary threshold: `lane_bin_th`
- Sliding window: `nwindows`, `margin`, `minpix`
- Steering gains: `DX_GAIN`, `DEGREE_GAIN`
- Speed: `STRAIGHT_VELO`, `TURN_VELO`, `TURN_FASTVEL`

---

## 🧪 Debug / Visualization
`driving.py` uses `cv2.imshow()` for debug windows (lane viewer, etc.).
On headless environments, disable GUI calls or use a virtual display.

---

