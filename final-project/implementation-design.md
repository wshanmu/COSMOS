---
layout: page
title: Implementation Design Hints
description: Suggested system architecture, data collection design, and command-line workflow for the final project.
parent: Final Project
nav_order: 1
permalink: /final-project/implementation-design/
---

# Implementation Design Hints

This page gives implementation hints for the final project. You will not receive a complete final-project codebase. Instead, use the previous labs as references and build your own system around a clean structure.

## Recommended Code Structure

Recommended starting structure:

```text
final_project/
  README.md
  src/
    collect.py
    combine_datasets.py
    extract_features.py
    train.py
    evaluate.py
    realtime_demo.py
    sensors/
      base_reader.py
      imu_reader.py
      uwb_reader.py
      mmwave_reader.py
      wifi_reader.py
      rfid_reader.py
  data/
    raw/
    processed/
  models/
  results/
    figures/
```

Good references from previous labs:

| Prior lab | Useful idea to reuse |
| --- | --- |
| IMU lab | Serial streaming, motion windows, smoothing, wearable placement |
| WiFi lab | Multi-device setup, CSI stream parsing, environment sensitivity |
| UWB lab | Trial-based collection, dataset folders, training and realtime evaluation |
| mmWave lab | Background subtraction, range profile features, point-cloud features |

## Data Collection Design

The most important part of the final project is not the classifier. It is collecting data in a way that makes the classification problem meaningful.

Use a coordinator-based design:

```text
start all sensor streams
record trial_start event timestamp
record trial_end event timestamp
ask whether to accept or reject the trial
save all raw data and event markers
segment the trials offline
```

The coordinator should own the official experiment clock. Use `time.monotonic()` in Python for timestamps. Do not try to make every sensor physically start at the exact same instant. Instead, start all sensors first, timestamp every sample on the laptop, and use event markers to cut the data into trials.

## Recommended Data Model

Each collection session should create one folder:

```text
data/raw/session_20260722_153012/
  session_metadata.json
  events.csv
  trials.csv
  imu.csv
  uwb.csv
  mmwave.csv
  wifi.csv
```

Example `events.csv`:

```csv
time_s,event,trial_id,gesture,collector,notes
0.000,session_start,,,,
6.231,trial_start,student01_pull_001,pull,student01,
9.231,trial_end,student01_pull_001,pull,student01,
```

Example sensor file:

```csv
time_s,sensor,ax,ay,az,gx,gy,gz
6.245,imu,0.01,-0.02,0.98,0.1,0.2,-0.1
6.266,imu,0.02,-0.01,0.99,0.0,0.3,-0.2
```

After collection, compute trial-relative time offline:

```python
trial_data = sensor_data[
    (sensor_data["time_s"] >= trial_start_time) &
    (sensor_data["time_s"] <= trial_end_time)
].copy()

trial_data["t_trial_s"] = trial_data["time_s"] - trial_start_time
```

## Sensor Reader Interface

Each sensor reader should hide sensor-specific details from `collect.py`.

Suggested interface:

```python
class SensorReader:
    def start(self, session_dir, session_t0):
        ...

    def stop(self):
        ...

    def close(self):
        ...
```

Each reader writes timestamped samples using the same session clock:

```python
t = time.monotonic() - session_t0
```

Then `collect.py` can treat IMU, UWB, mmWave, WiFi, and RFID streams in the same way.

## Gesture Protocols

For the required system, use a standard **3-second gesture action window**. This keeps data collection, feature extraction, and realtime evaluation simple.

Default trial timing:

```text
1 s prepare
1 s rest baseline
3 s gesture action
1 s cooldown
```

This works only if the gesture timing is controlled. For one-shot gestures, perform the action once near the center of the 3-second action window. For repeated gestures, repeat naturally throughout the 3-second action window.

## Discrete One-Shot Gestures

Examples:

- Pull
- Push
- Left
- Right
- Clockwise
- Anti-clockwise

Collect these as separate trials. Do not continuously repeat pull/push for a long recording, because the transitions may blend together and confuse the model.

## Continuous or Periodic Gestures

Examples:

- Clapping
- One-Arm Boxing
- Two-Arm Boxing
- Bye-Bye
- Making Fist and Open
- Palm Up-Down

For these gestures, repeat the motion naturally during the 3-second action window. If your group wants to collect longer continuous recordings, you must be careful during evaluation: split train/test by full recording or by collector, not by overlapping windows from the same recording.

## Example Config File

You can use a config file so you can change sensors, gestures, and durations without editing code.

Example `configs/imu_uwb.yaml`:

```yaml
session_name: imu_uwb_gesture_test
sensors:
  - imu
  - uwb

timing:
  duration: 3.0

collection:
  trials_per_gesture: 8
  require_accept: true

gestures:
  pull:
    type: discrete
  push:
    type: discrete
  clapping:
    type: periodic
  ...

ports:
  imu: /dev/cu.usbserial-IMU
  uwb_controller: /dev/cu.usbmodemCONTROLLER
  uwb_controlee: /dev/cu.usbmodemCONTROLEE
```

Windows groups can use COM ports in the same file:

```yaml
ports:
  imu: COM5
  uwb_controller: COM7
  uwb_controlee: COM8
```

## Desired Command-Line Workflow

Your exact commands may differ, but your project should support a similar workflow.

Check that selected sensors can stream and save:

```bash
python src/collect.py --config configs/imu_uwb.yaml --smoke-test --duration 10
```

Collect a small two-gesture dataset first:

```bash
python src/collect.py \
  --config configs/imu_uwb.yaml \
  --collector student01 \
  --gestures pull,push \
  --trials 3
```

Collect the main dataset:

```bash
python src/collect.py \
  --config configs/imu_uwb.yaml \
  --collector student01 \
  --gestures pull,push,left,right,clapping,t_arm \
  --trials 8
```

Combine datasets from group members:

```bash
python src/combine_datasets.py \
  data/raw/session_student01 \
  data/raw/session_student02 \
  data/raw/session_student03 \
  data/raw/session_student04 \
  --output data/processed/combined_group_dataset
```

Extract features:

```bash
python src/extract_features.py \
  data/processed/combined_group_dataset \
  --window-s 3.0 \
  --output data/processed/features_3s.csv
```

Train and evaluate:

```bash
python src/train.py data/processed/features_3s.csv --classifier random_forest
python src/evaluate.py models/random_forest_YYYYMMDD_HHMMSS.joblib --split by_collector
```

Run a realtime demo:

```bash
python src/realtime_demo.py \
  --config configs/imu_uwb.yaml \
  --model models/random_forest_YYYYMMDD_HHMMSS.joblib \
  --window-s 3.0 \
  --vote-window 5
```

## Development Roadmap

Build the system step by step.

1. Start one sensor and save raw data.
2. Start two sensors at the same time and save both streams.
3. Add `events.csv` and verify trial markers.
4. Implement one discrete gesture, such as `pull`.
5. Collect two gestures and train a first classifier.
6. Add the second sensor features.
7. Compare single-sensor baseline vs fused model.
8. Add more gestures.
9. Add realtime prediction with majority voting.
10. Prepare the poster and demo.
