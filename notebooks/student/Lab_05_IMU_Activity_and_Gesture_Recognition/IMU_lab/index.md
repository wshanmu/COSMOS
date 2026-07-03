---
layout: page
title: Lab 5 IMU Student Code
permalink: /notebooks/student/Lab_05_IMU_Activity_and_Gesture_Recognition/IMU_lab/
nav_exclude: true
---

# Lab 5 IMU Student Code

Download or copy this folder before starting Lab 5. The ESP32 firmware is pre-flashed; these files are for serial visualization, smoothing, trajectory capture, and simple motion classification.

## Setup

From this folder:

```bash
cd tools
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Windows PowerShell:

```powershell
cd tools
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

## Files

- [requirements.txt]({{ site.baseurl }}/notebooks/student/Lab_05_IMU_Activity_and_Gesture_Recognition/IMU_lab/tools/requirements.txt)
- [plot_imu_serial.py]({{ site.baseurl }}/notebooks/student/Lab_05_IMU_Activity_and_Gesture_Recognition/IMU_lab/tools/plot_imu_serial.py)
- [plot_imu_smoothed_TODO.py]({{ site.baseurl }}/notebooks/student/Lab_05_IMU_Activity_and_Gesture_Recognition/IMU_lab/tools/plot_imu_smoothed_TODO.py)
- [capture_trajectory.py]({{ site.baseurl }}/notebooks/student/Lab_05_IMU_Activity_and_Gesture_Recognition/IMU_lab/tools/capture_trajectory.py)
- [capture_trajectory_smoothed_TODO.py]({{ site.baseurl }}/notebooks/student/Lab_05_IMU_Activity_and_Gesture_Recognition/IMU_lab/tools/capture_trajectory_smoothed_TODO.py)
- [IMU classifier README]({{ site.baseurl }}/notebooks/student/Lab_05_IMU_Activity_and_Gesture_Recognition/IMU_lab/tools/IMU_Classifier/README.md)
- [python_imu_segment_demo_student.py]({{ site.baseurl }}/notebooks/student/Lab_05_IMU_Activity_and_Gesture_Recognition/IMU_lab/tools/IMU_Classifier/python_imu_segment_demo_student.py)
- [run_student.sh]({{ site.baseurl }}/notebooks/student/Lab_05_IMU_Activity_and_Gesture_Recognition/IMU_lab/tools/IMU_Classifier/run_student.sh)
- [classifier requirements-python.txt]({{ site.baseurl }}/notebooks/student/Lab_05_IMU_Activity_and_Gesture_Recognition/IMU_lab/tools/IMU_Classifier/requirements-python.txt)

Ask a TA before modifying or reflashing the ESP32 firmware.
