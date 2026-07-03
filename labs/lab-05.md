---
layout: page
title: "Lab 5: IMU Activity and Gesture Recognition"
description: IoT lab on ESP32/BMI270 IMU streaming, smoothing, trajectory estimation, and motion classification.
parent: Labs
nav_order: 5
---

# Lab 5: IMU Activity and Gesture Recognition **IoT**{: .label .label-iot }

Date: July 10  
TA: Shanmu Wang  
Hardware: ESP32 board with BMI270 IMU  
Total time: About 4 hours

## Goals

By the end of this lab, each group should be able to:

- explain what an IMU measures and why accelerometer readings include gravity;
- stream 6-axis IMU readings from an ESP32 over USB serial;
- identify the physical x/y/z axes of the IMU board from accelerometer and gyroscope plots;
- implement a moving-average filter and explain the noise-latency tradeoff;
- explain why dead-reckoned trajectories from IMU double integration drift quickly;
- compare raw and smoothed trajectory estimates;
- collect short labeled IMU motion segments and train a simple live classifier.

## Lab Code

Use the public IMU lab repository as the source of truth:

[https://github.com/wshanmu/IMU_lab](https://github.com/wshanmu/IMU_lab)

Clone it once:

```bash
git clone https://github.com/wshanmu/IMU_lab.git
cd IMU_lab
```

Before lab, update to the latest version:

```bash
git pull
```

If `git pull` reports local changes, ask a TA before running reset, checkout, or other commands that overwrite files.

The main files used in this lab are:

| File | Purpose |
| --- | --- |
| `tools/plot_imu_serial.py` | Live raw accelerometer and gyroscope plot. |
| `tools/plot_imu_smoothed_TODO.py` | Student TODO for moving-average smoothing. |
| `tools/capture_trajectory.py` | Captures a short motion and estimates a 2D XY trajectory. |
| `tools/capture_trajectory_smoothed_TODO.py` | Student TODO for adding smoothing before trajectory estimation. |
| `tools/IMU_Classifier/python_imu_segment_demo_student.py` | Student TODO for a simple 3-class IMU classifier. |

The firmware has already been flashed for this lab. Students only need the Python tools unless a TA asks them to rebuild or reflash the ESP32.

## System Overview

The lab device uses an ESP32 as the microcontroller and a BMI270 as the IMU sensor. The ESP32 communicates with the BMI270 over I2C, then prints each accelerometer and gyroscope sample over USB serial.

```text
BMI270 IMU -> I2C -> ESP32 -> USB serial -> Python visualization scripts
```

The firmware streams lines like:

```text
accel[g] x= 0.012 y=-0.034 z= 0.998 | gyro[dps] x= 0.10 y=-0.20 z= 0.05
```

In the prepared firmware, the IMU is configured for approximately 100 Hz streaming, accelerometer range `+/-4 g`, and gyroscope range `+/-1000 dps`. The Python scripts parse the serial text and plot the six channels in real time.

## ESP-IDF Setup

The ESP32 firmware is pre-flashed, so this setup is mainly for understanding the toolchain and for recovery if a board needs to be reprogrammed.

1. Install or update [Visual Studio Code](https://code.visualstudio.com/).
2. In VS Code, install the official **ESP-IDF** extension.
3. Open the command palette:
   - macOS: `Command+Shift+P`
   - Windows/Linux: `Ctrl+Shift+P`
4. Run `ESP-IDF: Open ESP-IDF Installation Manager`.
5. Install ESP-IDF and its tools using the installer.
6. Run `ESP-IDF: Select Current ESP-IDF Version`.
7. Run `ESP-IDF: Doctor Command` and fix any reported setup issues.
8. As a sanity check, run `ESP-IDF: New Project`, create a Hello World project, then build, flash, and monitor it.

Common troubleshooting:

- update VS Code before installing the ESP-IDF extension;
- use a USB data cable, not a charge-only cable;
- close any serial monitor before running a Python script, because only one program can open the port at a time;
- select target `esp32` for the lab board;
- avoid paths with spaces if using an older ESP-IDF version;
- if flashing fails, unplug and reconnect the board, then select the serial port again.

Official references:

- [ESP-IDF Extension for VS Code installation guide](https://docs.espressif.com/projects/vscode-esp-idf-extension/en/latest/installation.html)
- [ESP-IDF Get Started guide for ESP32](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html)
- [Bosch BMI270 product page](https://www.bosch-sensortec.com/en/products/motion-sensors/imus/bmi270)

## Python Setup

Open a terminal in the lab code folder:

```bash
cd IMU_lab/tools
```

Create a Python environment if you have not already done so.

macOS/Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Find the ESP32 serial port.

macOS:

```bash
ls /dev/cu.usb* /dev/cu.SLAB* /dev/cu.wch* 2>/dev/null
```

Windows:

```text
Open Device Manager -> Ports (COM & LPT), then look for the new COM port.
```

Use your actual port in the commands below. Examples:

- macOS: `/dev/cu.usbserial-5B1F0080901`
- Windows: `COM5`

## Four-Hour Plan

### 0:00-0:20 - Demo and IMU Concepts

The TA will first show the live visualization and a working classifier demo.

An inertial measurement unit, or IMU, usually contains:

- **Accelerometer:** measures specific force along x/y/z. When the board is stationary, one or more axes still measure gravity.
- **Gyroscope:** measures angular velocity around x/y/z.

Useful checks:

- If the board is still, the acceleration magnitude should be close to `1 g`.
- If one axis points upward, that accelerometer channel should be close to `+1 g` or `-1 g`.
- If you rotate around one board axis, the matching gyroscope channel should have the largest response.

### 0:20-0:45 - Visualize Raw 6-Axis Data

Run the live plot:

```bash
python plot_imu_serial.py --port YOUR_PORT --baud 115200
```

Replace `YOUR_PORT` with your serial port.

Do these tests:

1. Hold the board still in at least three orientations.
2. Move the board back and forth along one physical direction.
3. Rotate the board around each physical axis.
4. Watch which accelerometer or gyroscope channel changes the most.

Deliverable for this step: fill in a table like this.

| Physical motion | Channel with strongest response | Evidence from plot |
| --- | --- | --- |
| Board x direction |  |  |
| Board y direction |  |  |
| Board z direction |  |  |
| Rotation around x |  |  |
| Rotation around y |  |  |
| Rotation around z |  |  |

### 0:45-1:25 - Implement Moving-Average Smoothing

Raw IMU signals are noisy because of MEMS sensor noise, hand tremor, quantization, board vibration, and small timing variation. A moving average reduces high-frequency noise by replacing each new sample with the average of the most recent `N` samples.

For a window of size `N`:

```text
smoothed[t] = mean(raw[t-N+1], ..., raw[t])
```

In `plot_imu_smoothed_TODO.py`, implement the `MovingAverageFilter.update()` method. The intended data structure is:

- a `deque` that stores the most recent samples;
- a `running_sum` that avoids recomputing the full sum every update;
- one update operation that appends the newest value, removes the oldest value if the window is too long, and returns the current average.

Run your version:

```bash
python plot_imu_smoothed_TODO.py --port YOUR_PORT --baud 115200 --average-window 5
```

Try at least three window sizes, such as `3`, `10`, and `30`.

Checkpoint questions:

- Which window size removes the most visible noise?
- Which window size creates the most delay?
- Why is too much smoothing bad for fast gestures?

### 1:25-2:15 - Capture a Short Trajectory

In principle, a trajectory can be estimated from acceleration:

```text
acceleration -> integrate once -> velocity -> integrate again -> position
```

In practice, this is difficult. Gravity is about `9.8 m/s^2`, while many hand motions create smaller linear accelerations. A small gravity-removal error or sensor bias becomes a growing velocity error after integration, then a larger position error after the second integration.

Run the prepared trajectory demo:

```bash
python capture_trajectory.py --port YOUR_PORT --baud 115200 --capture-seconds 5
```

Keep the board still for one second, press `c`, then perform a short motion while the script captures data. Try:

- a short straight-line motion;
- a small square;
- a small circle;
- a motion that starts and ends at the same place.

What the script is doing:

1. Uses the pre-capture samples as a baseline for gyro bias and gravity direction.
2. Uses gyroscope readings to update orientation during the capture.
3. Rotates acceleration into a world frame.
4. Subtracts gravity.
5. Applies small deadbands and damping to reduce obvious drift.
6. Integrates acceleration to velocity and velocity to position.
7. Plots the reconstructed XY path.

Thinking question: why does this lab visualize only the 2D XY trajectory instead of a full 3D path?

### 2:15-2:50 - Add Smoothing Before Trajectory Estimation

Copy the moving-average idea from `plot_imu_smoothed_TODO.py` into `capture_trajectory_smoothed_TODO.py`.

Run:

```bash
python capture_trajectory_smoothed_TODO.py --port YOUR_PORT --baud 115200 --capture-seconds 5 --average-window 5
```

Compare the raw and smoothed trajectories for the same type of motion.

Deliverable for this step:

- one screenshot or sketch of a raw trajectory;
- one screenshot or sketch of a smoothed trajectory;
- a short answer: does smoothing solve trajectory drift, or does it only reduce part of the noise?

Expected observation: smoothing can make the path less jittery, but it cannot remove constant bias, imperfect gravity subtraction, orientation error, or integration drift. A very large window can also lag behind the real motion.

### 2:50-3:40 - Train a Simple IMU Motion Classifier

Open the classifier folder:

```bash
cd IMU_Classifier
python -m pip install -r requirements-python.txt
```

Run the student version:

macOS/Linux:

```bash
./run_student.sh --port YOUR_PORT --classes Still,Shake,Turn
```

Windows PowerShell:

```powershell
python python_imu_segment_demo_student.py --port YOUR_PORT --classes Still,Shake,Turn
```

You may choose a different three-class task set, such as:

- `Still,Shake,Turn`
- `Tap,Circle,Flip`
- `UpDown,LeftRight,Rotate`

Workflow:

1. Select a class.
2. Press `c`, `Space`, `Enter`, or the **Capture 1 s** button.
3. Perform the selected motion for the next second.
4. Collect at least five examples per class if time allows.
5. Press `t` or **Train**.
6. Perform new examples and observe the live prediction.

Student TODO: in `python_imu_segment_demo_student.py`, add summary statistics to the feature vector. Good first features are:

- standard deviation of each channel;
- min and max of each channel;
- range of each channel;
- RMS energy of each channel;
- mean absolute first difference of each channel.

Discussion questions:

- Which three motions are easiest to separate?
- Does the classifier fail when a different student performs the same motion?
- Does adding summary features make predictions more stable?

### 3:40-4:00 - Checkoff and Cleanup

Each group should show the TA:

1. live raw IMU data from the ESP32;
2. the completed axis-response table;
3. moving-average smoothing running with at least two window sizes;
4. one trajectory capture and one explanation of why it drifts;
5. the smoothed trajectory comparison;
6. one trained 3-class live classifier.

## Final Questions

Answer these briefly in your group notes:

1. What does the accelerometer measure when the board is completely still?
2. How did you identify the physical x/y/z axes?
3. What is the tradeoff when increasing the moving-average window size?
4. Why does double integration of IMU acceleration drift so quickly?
5. Why is the XY trajectory easier to interpret than the full 3D trajectory in this lab?
6. Which features helped the classifier distinguish your motions?

## Troubleshooting

| Problem | Likely Cause | Fix |
| --- | --- | --- |
| No serial data | Wrong port, charge-only cable, or board not powered | Try another USB cable, check Device Manager or `/dev/cu.*`, reconnect the board. |
| Port is busy | Another program has the serial port open | Close ESP-IDF monitor, Arduino serial monitor, PuTTY, or another Python script. |
| Plot opens but stays flat | Board is not streaming expected text format | Ask the TA to confirm the firmware and baud rate. |
| `ModuleNotFoundError` | Python packages not installed in the active environment | Activate the environment and run `python -m pip install -r requirements.txt`. |
| Qt or PyQt error | GUI backend issue | Restart the terminal, confirm `PyQt6` installed, or ask a TA for the backup computer. |
| Classifier predictions unstable | Too few examples or motions too similar | Collect more examples and choose more distinct motions. |
