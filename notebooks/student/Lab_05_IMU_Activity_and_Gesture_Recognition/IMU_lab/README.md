# Lab 5 IMU Student Code

This folder contains the student-facing Python tools for Lab 5.

## Setup

From this folder:

```bash
cd tools
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

On Windows PowerShell:

```powershell
cd tools
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

## Main Scripts

- `tools/plot_imu_serial.py`: live raw IMU plot.
- `tools/plot_imu_smoothed_TODO.py`: moving-average smoothing TODO.
- `tools/capture_trajectory.py`: short trajectory capture demo.
- `tools/capture_trajectory_smoothed_TODO.py`: smoothed trajectory TODO.
- `tools/IMU_Classifier/python_imu_segment_demo_student.py`: 3-class motion classifier TODO.

The ESP32 firmware is pre-flashed for the lab. Ask a TA before changing the board firmware.
