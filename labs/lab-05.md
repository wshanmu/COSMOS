---
layout: page
title: "Lab 5: IMU Activity and Gesture Recognition"
description: IoT lab on IMU data collection, visualization, feature extraction, and motion classification.
parent: Labs
nav_order: 5
---

# Lab 5: IMU Activity and Gesture Recognition **IoT**{: .label .label-iot }

Date: July 10  
TA: Shanmu Wang  
Hardware: phone IMU or TA-provided IMU board  
Total time: About 4 hours

## Goals

By the end of this lab, each group should be able to:

- explain what accelerometers and gyroscopes measure;
- identify IMU axes and sampling-rate issues;
- collect labeled motion data from an IMU sensor;
- visualize raw acceleration and angular velocity signals;
- segment sensor streams into sliding windows;
- extract time-domain and frequency-domain features;
- train and evaluate a simple motion classifier;
- explain why activity recognition models may fail on new users or new sensor placements.

## Hardware and Software

Use whichever IMU capture setup the TA provides:

| Option | Description |
| --- | --- |
| Smartphone IMU | A phone sensor logger exports timestamped accelerometer and gyroscope CSV files. |
| IMU development board | A board streams accelerometer and gyroscope values over serial, BLE, WiFi, or USB. |
| Pre-recorded backup data | Use this if live collection fails or a sensor is unavailable. |

Required data columns:

`timestamp_s, ax, ay, az, gx, gy, gz, label, trial_id, subject_id`

Recommended units:

- `ax, ay, az`: acceleration in `m/s^2` or `g`;
- `gx, gy, gz`: angular velocity in `deg/s` or `rad/s`;
- `timestamp_s`: seconds from the start of the recording.

Keep the units consistent across all trials.

## Group Roles

Each group should assign:

- Sensor lead: runs the logger and checks axes/sampling rate.
- Protocol lead: reads the trial script and keeps timing consistent.
- Data lead: names files, checks labels, and merges CSVs.
- Modeling lead: builds plots, features, and classifier.

## Before Recording

1. Decide the sensor placement:
   - phone in hand;
   - phone in pocket;
   - phone strapped to wrist;
   - IMU board held in hand.
2. Keep the placement fixed for the required dataset.
3. Record the sensor orientation. For example: screen facing up, top of phone pointing forward.
4. Set the sampling rate to 50-100 Hz if the logger allows it.
5. Do a 10-second test recording and confirm all columns are present.

Do not mix placements during the required trials. A model trained on wrist data and tested on pocket data is solving a different problem.

## Part A: IMU Concepts and Axis Check

### 0:00-0:20 - Mini-Lesson

An IMU usually contains:

- **Accelerometer:** measures specific force along x/y/z. When stationary, it still measures gravity.
- **Gyroscope:** measures angular velocity around x/y/z.
- **Magnitude:** removes axis sign/orientation partially:

`acc_mag = sqrt(ax^2 + ay^2 + az^2)`

`gyro_mag = sqrt(gx^2 + gy^2 + gz^2)`

Important limitations:

- accelerometer readings include gravity;
- gyroscope integration drifts over time;
- sampling rates may not be perfectly constant;
- axes depend on sensor placement and orientation;
- hand motions vary across people.

### 0:20-0:35 - Axis Calibration

Run three quick checks:

1. Place the sensor flat and still for 5 seconds.
2. Rotate it slowly around one axis.
3. Shake it briefly along one axis.

Plot `ax, ay, az, gx, gy, gz` for this test.

Checkpoint questions:

- Which axis is most aligned with gravity when the sensor is flat?
- Which gyro axis changes most during your rotation?
- Does the logger report values at a roughly constant sampling interval?

## Part B: Required Motion Dataset

### 0:35-1:25 - Collect Labeled Trials

Collect five labels. Each trial should last 10 seconds.

| Label | Trial Script | Trials |
| --- | --- | --- |
| `stationary` | Hold the sensor still. | 4 |
| `walking` | Walk normally in a straight path or loop. | 4 |
| `shake` | Shake the sensor with short rapid motions. | 4 |
| `raise_lower` | Raise and lower the hand/phone repeatedly. | 4 |
| `circle` | Move the sensor in a circular motion. | 4 |

File naming:

`group##_subject##_label##_trial##.csv`

Examples:

- `group03_subject01_walking_trial02.csv`
- `group03_subject02_circle_trial04.csv`

Rules:

- Start each trial with 2 seconds of stillness if possible.
- Avoid talking while recording if audio/video apps are open.
- Keep distance from other groups so labels do not get mixed.
- Save raw data before doing any filtering or trimming.

### 1:25-1:45 - Data Quality Check

Before modeling, check:

- each file has a label;
- each trial is about 10 seconds;
- timestamps are increasing;
- there are no long gaps in sampling;
- acceleration and gyroscope columns are numeric;
- no label has fewer than 3 usable trials.

Make two plots:

1. Raw `ax, ay, az` for one trial from each label.
2. `acc_mag` and `gyro_mag` for one trial from each label.

Discussion prompt: Which labels look easiest to separate by eye? Which labels overlap?

## Part C: Sliding Windows and Features

### 1:45-2:25 - Segment the Data

Split each trial into sliding windows.

Recommended settings:

- window length: 2 seconds;
- overlap: 50%;
- minimum valid samples per window: 75% of expected samples.

Each window inherits the label of its trial.

Avoid data leakage:

- Split train/test by `trial_id` or `subject_id`.
- Do not randomly split individual windows if windows from the same trial can appear in both train and test.

### 2:25-2:55 - Extract Features

For each window, compute features from:

- `ax, ay, az`;
- `gx, gy, gz`;
- `acc_mag`;
- `gyro_mag`.

Required features:

- mean;
- standard deviation;
- minimum and maximum;
- range: `max - min`;
- median absolute deviation;
- signal energy: `mean(x^2)`;
- dominant frequency for `acc_mag`;
- total variation: `sum(abs(diff(x)))`.

Optional features:

- correlation between axes;
- zero-crossing rate after subtracting the mean;
- spectral entropy;
- ratio of low-frequency to high-frequency energy.

Feature table format:

`window_id, subject_id, trial_id, label, feature_1, feature_2, ...`

Checkpoint: print the shape of your feature table and verify that each label has multiple windows.

## Part D: Classification

### 2:55-3:35 - Train and Evaluate

Train at least two classifiers:

- k-nearest neighbors;
- logistic regression;
- decision tree.

Recommended evaluation:

1. Split by trial: 70% of trials for training, 30% for testing.
2. Standardize features using training-set statistics only.
3. Train the models.
4. Report accuracy and confusion matrix.
5. Compare which labels are confused.

Required plots:

- confusion matrix;
- bar chart of per-class accuracy;
- 2D PCA or t-SNE plot of feature windows if time allows.

Questions:

- Which model performed best?
- Which labels were hardest to classify?
- Did raw axes or magnitude features seem more useful?
- Would the model work if the phone were placed in a pocket instead of held in hand?

### 3:35-3:50 - Cross-Person or New-Trial Test

Choose one:

**Option 1: New trial test**

Record one new trial for each label and test your trained model.

**Option 2: Cross-person test**

Train on one subject and test on another subject.

Record:

- accuracy on normal train/test split;
- accuracy on new-trial or cross-person split;
- one reason performance changed.

## Part E: Sensor Fusion Extension

### 3:50-4:00 - Complementary Filter Demo

If time remains, estimate pitch or roll.

Accelerometer-only estimate:

`roll_acc = atan2(ay, az)`

Gyroscope integration:

`roll_gyro[t] = roll_gyro[t-1] + gx[t] * dt`

Complementary filter:

`roll[t] = alpha * (roll[t-1] + gx[t] * dt) + (1 - alpha) * roll_acc[t]`

Try `alpha = 0.98`.

Discussion:

- Why does the gyro-only angle drift?
- Why is the accelerometer angle noisy during fast motion?
- Why does the complementary filter improve the estimate?

## Deliverables

Submit one short report per group with:

1. Sensor setup: device, placement, sampling rate, and orientation.
2. Data collection table: labels, number of trials, duration.
3. Raw signal plots for at least three labels.
4. Magnitude plots for at least three labels.
5. Feature table summary: number of windows and number of features.
6. Classifier results: accuracy and confusion matrix.
7. New-trial or cross-person test result.
8. Answers to the discussion questions.

## What to Conclude

A strong conclusion should mention:

- IMU activity recognition depends heavily on placement, orientation, and user behavior.
- Magnitude features can reduce orientation sensitivity but do not solve every problem.
- Sliding-window choices affect both accuracy and latency.
- Random window splits can overestimate performance if windows from the same trial appear in train and test.
- Sensor fusion is useful because accelerometers and gyroscopes have complementary failure modes.

## Extension Ideas

If your group finishes early:

- build a live classifier that predicts the current label every 1-2 seconds;
- compare 1-second, 2-second, and 4-second windows;
- train on one placement and test on another;
- remove gyroscope features and measure how accuracy changes;
- design your own gesture vocabulary and collect a small dataset;
- try a simple final-project idea using IMU plus another sensor.
