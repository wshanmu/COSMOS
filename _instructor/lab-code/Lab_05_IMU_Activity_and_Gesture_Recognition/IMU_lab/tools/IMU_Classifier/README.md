# IMU Classifier

This folder contains a Python GUI for 3-task IMU classification. It reads BMI270 data from the ESP32 serial port, records 1-second IMU windows, extracts features, and trains a simple linear SVM classifier.

Each training example is one 1-second segment containing:

- accelerometer: `ax`, `ay`, `az` in `g`
- gyroscope: `gx`, `gy`, `gz` in `deg/s`
- one label from three task classes

The default classes are `Task 1`, `Task 2`, and `Task 3`. You can rename them from the command line.

## Files

- `python_imu_segment_demo.py`: full instructor/reference version. It uses the resampled IMU waveform plus summary statistics as features.
- `python_imu_segment_demo_student.py`: student version. It uses only the resampled IMU waveform and leaves the statistics features as TODOs.
- `run_python.sh`: launches the full version.
- `run_student.sh`: launches the student version.
- `requirements-python.txt`: Python dependencies for this folder.

## Serial Input Format

The ESP32 should print lines like:

```text
accel[g] x= 0.012 y=-0.034 z= 0.998 | gyro[dps] x= 0.10 y=-0.20 z= 0.05
```

This matches the BMI270 logging format used in the ESP32 code.

## Setup

From `tools/IMU_Classifier`:

```bash
python3 -m pip install -r requirements-python.txt
```

## Run With ESP32

Find the ESP32 serial port:

```bash
ls /dev/cu.*
```

Run the full reference version:

```bash
./run_python.sh --port /dev/cu.usbserial-5B1F0080901
```

Run the student TODO version:

```bash
./run_student.sh --port /dev/cu.usbserial-5B1F0080901
```

Use custom class names if your three tasks are known:

```bash
./run_python.sh --port /dev/cu.usbserial-5B1F0080901 --classes Still,Shake,Turn
```

Test the GUI without hardware:

```bash
./run_python.sh --demo-signal
```

## Student Workflow

1. Pick three tasks, for example `Still`, `Shake`, and `Turn`.
2. Select the class in the dropdown.
3. Press `c`, `Space`, `Enter`, or the `Capture 1 s` button.
4. Perform that task for the next 1 second.
5. Collect at least two examples for each class.
6. Press `t` or `Train`.
7. Perform a task again and watch the live classification.
8. Press `s` to save the trained model, or `l` to load it later.

## Controls

- `Down`: select the next class
- `c`, `Space`, or `Enter`: record the next 1-second segment
- `t`: train the classifier, or switch back to collection mode
- `s`: save the trained model
- `l`: load the saved model
- `b`: toggle detection logging

## What The Full Version Does

For each 1-second segment, `python_imu_segment_demo.py`:

1. Resamples the segment to a fixed number of samples, default `20`.
2. Centers each IMU channel by subtracting its mean.
3. Uses the centered six-channel waveform as features.
4. Adds summary features: mean, standard deviation, min, max, range, RMS, and first-difference statistics.
5. Trains a linear SVM classifier with `scikit-learn`.
6. Uses a small voting window so the displayed class is more stable.

## Student TODO

In `python_imu_segment_demo_student.py`, find the `STUDENT TODO` block inside `ImuSegmentFeatureExtractor.capture_instance`.

The student version currently uses:

```python
measurements = centered.T.reshape(-1).astype("float32")
```

Students can improve the classifier by computing extra statistics and concatenating them:

```python
summary = self.np.concatenate([...]).astype("float32")
measurements = self.np.concatenate([centered.T.reshape(-1), summary]).astype("float32")
```

Suggested statistics:

- standard deviation of each channel
- min and max of each channel
- range of each channel
- RMS energy of each channel
- mean absolute first difference of each channel

Useful options:

```bash
./run_python.sh --port /dev/cu.usbserial-5B1F0080901 --segment-seconds 1
./run_python.sh --port /dev/cu.usbserial-5B1F0080901 --target-samples 20
./run_python.sh --port /dev/cu.usbserial-5B1F0080901 --min-samples-per-class 3
./run_python.sh --port /dev/cu.usbserial-5B1F0080901 --model my_imu_model.joblib
```

## Notes

- Keep each task motion consistent while collecting training data.
- Make sure every class has a similar number of examples.
- If predictions are unstable, collect more examples or make the three tasks more different.
- Saved models only work with the same class names and feature settings used during training.
