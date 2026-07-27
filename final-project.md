---
layout: page
title: Final Project
description: Final project instructions for UCLA COSMOS Cluster 10.
nav_order: 5
permalink: /final-project/
has_children: true
---

# Final Project

The final project is an end-to-end IoT and data science system. Each group will collect real sensor data, build a machine learning pipeline, and demonstrate a gesture recognition system.

[Final project group signup form](https://docs.google.com/spreadsheets/d/1wGb3KYH4sK1ysYYhA1DYE0sdy-Vcq1SoBPUdvngxoZw/edit?usp=sharing){: .btn .btn-primary }
[Demo Gesture Video on Box](https://ucla.app.box.com/file/2361182315011?s=aa9zjcqj5po2sgvi1b2zjyoez1ehkimw){: .btn }
[Implementation Design Hints]({{ site.baseurl }}/final-project/implementation-design/){: .btn }

## Project Goal

Build a system that classifies as many of the provided gestures as possible.

The strongest projects should do more than run one sensor and one model. You should design a complete sensing pipeline:

1. choose useful sensors from the kit;
2. design the setup such that the gestures create different patterns;
3. collect a balanced dataset;
4. extract features or prepare model inputs;
5. train and compare machine learning models;
6. evaluate the model carefully;
7. demonstrate live or near-real-time gesture classification.

Your system should use at least two sensing modalities unless the TA approves a single-sensor design. A good project compares single-sensor baselines against a fused model, then explains whether sensor fusion helped.

## Groups

Form a group of 4 and fill in the group signup sheet.

After receiving your project box:

1. record your group members in the sheet;
2. record the color label of your box in the sheet;
3. check that all components are present;
4. ask Shanmu if you need extra cables, RFID tags, tape, batteries, or other small materials.

## Project Kit

Each group will receive one color-labeled box with the following basic components:

| Item | Quantity | Notes |
| --- | ---: | --- |
| IMU device | 1 | ESP32 Core2 module |
| ESP32-CAM | 1 | For the WiFi CSI Extraction |
| ESP32-C3 | 1 | For the WiFi CSI Extraction |
| mmWave radar | 1 | Radar range profile and point-cloud sensing |
| RFID reader | 1 | RFID-based interaction/sensing |
| UWB modules | 3 | Distance/ranging-based sensing |
| Strap | 2 | For wearable or fixed mounting |
| 4-port USB hub | 1 | For connecting multiple devices |
| USB-to-Type-C converter | 1 | Adapter |
| 6-ft Micro-USB cable | 2 | For ESP32-CAM / UWB modules |
| 6-ft Type-C cable | 2 | For IMU / RFID Reader |

These are the base components. You may use additional simple materials if needed.

## Gesture List

Use the following gesture set. Keep the class names consistent in your code and notes.

1. Pull
2. Push
3. Clockwise
4. Anti-clockwise
5. Right
6. Left
7. Bye-Bye
8. One-Arm Boxing
9. Clapping
10. Two-Arm Boxing
11. T-Arm
12. Raise Arms
13. Soli
14. Making Fist and Open
15. Palm Up-Down

Start with a smaller subset that your group can collect reliably, then expand. A practical strategy is to first make 4 gestures work well, then add more gestures once the data pipeline and evaluation are stable.

## Minimum System Requirements

Your final project should include:

1. A clear sensing setup.
2. A data collection protocol.
3. Data from multiple group members when possible.
4. At least two sensing modalities.
5. A single-sensor baseline model.
6. A fused or improved model.
7. A confusion matrix.
8. A discussion of which gestures are easy or hard to classify.
9. A live or recorded demo.

For implementation guidance, use the separate design page:

[Open implementation design hints]({{ site.baseurl }}/final-project/implementation-design/){: .btn }

## Modeling Requirements

You should compare at least two models or two feature sets.

Examples:

- KNN vs SVM;
- random forest vs SVM;
- raw time-window features vs handcrafted features;
- single-sensor model vs fused-sensor model;
- early fusion vs late fusion.

For sensor fusion, choose one of these approaches:

| Fusion type | Meaning |
| --- | --- |
| Early fusion | Combine features from multiple sensors into one feature vector, then train one model. |
| Late fusion | Train one model per sensor, then combine their predictions by voting or averaging probabilities. |

## Evaluation Requirements

Report more than one number. At minimum, include:

1. overall accuracy;
2. confusion matrix;
3. per-class observations;
4. comparison between your baseline and improved model;
5. one short explanation of the largest failure mode.

If you have data from multiple people, also evaluate one person as the held-out test user. This is more realistic than randomly mixing trials from every person into both training and testing.

Example evaluation questions:

- Which gestures are most often confused?
- Does the model still work on a person who was not in the training set?
- Did adding a second sensor improve accuracy?
- Did the second sensor help all gestures, or only some?

## Final Poster and Demo Presentation

Poster submission is due by 12:00 PM on Wednesday, July 29.

Your final poster should at least show:

1. the sensing setup;
2. the gestures you attempted;
3. the dataset size;
4. your model pipeline;
5. your evaluation results;
6. one limitation and one future improvement.

Final demos and presentations will take place on Saturday, August 1, during the morning session ending at 12:00 PM at the Grand Horizons Ballroom.

## Practical Tips

- **Build the system incrementally.**
  - Keep the physical setup repeatable.
  - Balance the number of trials per gesture.
  - Use clear file and folder names.
  - Save raw data and processed features separately.
  - Run a small training test early, even if the first dataset is messy.
  - Add gestures only after your evaluation code works.
  - Prefer a reliable 6-gesture system over an unreliable 15-gesture system with no analysis.

- **Check data quality before collecting more data.**
  - In machine learning, "garbage in, garbage out" is a real problem. If the training data is noisy, mislabeled, or inconsistent, the model will almost certainly perform poorly.
  - After each collection session, you can inspect the data before collecting more trials.
  - For example, for IMU data, plot the 3-axis acceleration signals. For gestures such as push and pull, check whether at least one axis shows the expected acceleration pattern.
  - For UWB data, plot the ranging signal and check whether the distance changes match the gesture.
  - For mmWave data, you can run the near-field visualization script from the mmWave lab before collecting data. Use it to decide where the radar should point and how the gesture should be performed so that different gestures produce clearer and more distinguishable patterns.

- **Make sure the update rate is sufficient.**
  - The sensor update rate must be high enough to capture the motion you want to classify.
  - For example, 5 Hz UWB ranging is usually too slow to classify fast hand gestures reliably.
  - The IMU may not reliably sustain a 100 Hz update rate when many USB devices are connected at the same time. If the timestamps or values look irregular, lower the IMU sampling rate, for example to 50 Hz or 25 Hz.

- **Run smoke tests for multi-sensor setups.**
  - Before collecting labeled trials, confirm that every selected sensor can stream and save data.
  - If hardware stops responding, first unplug and reconnect the device.
  - Then check cable connections, including both sides of the mmWave radar wiring, the USB-to-UART adapter, and whether the UWB board shows the expected green status light.
  - To check whether it is the hardware issue or software issue, first run the unit-test code from each lab (e.g., `get_range_profile` from the mmWave lab)
  - Avoid pointing your mmWave radar directly at another group's setup.
  - Use the channel and preamble code assigned to your group in the Google Sheet to reduce UWB interference.

- **Debug Tips for UWB boards.**
  - Make sure you are using the correct Python environment. In the UWB lab, you used a Python 3.9 environment and installed the required packages there.
  - If your group's data collection script does not work, first return to the UWB lab's `run_fira_twr` example and confirm that each board can range successfully.
  - Test the UWB setup with only two UWB boards connected directly to the laptop first. Confirm that `run_fira_twr` works, then gradually add other sensors such as the IMU and mmWave radar.
  - If the UWB boards work when plugged directly into the laptop but fail through the USB hub, ask Shanmu for help.
  - If all boards work with `run_fira_twr` but fail only when running your group's data collection script, the likely issue is in your code or command-line parameters.
  - Some groups may want to use 3 UWB nodes. The intended setup is 1 controller and 2 controlees. The UWB lab code only supports 1 controller and 1 controlee, so you will need to read the device documentation and adjust command-line parameters such as slot number and ranging schedule.

- **Evaluate in stages.**
  - A random train/test split is the easiest setting and often gives the highest accuracy.
  - Cross-user evaluation is harder and more realistic.
  - Real-time evaluation is usually the hardest because it adds streaming, timing, and windowing issues.
  - If your random-split accuracy is already poor, the real-time demo is unlikely to work well.
  - Report more than final overall accuracy. For multi-class classification, include a confusion matrix and per-class precision/recall or F1-score. If you include a `rest` or `unknown` class, also discuss false positives and false negatives.

- **Use ablation studies to justify sensor fusion.**
  - Compare single-sensor baselines against fused models, such as IMU only, UWB only, and IMU+UWB.
  - Report whether fusion improves all gestures or mainly helps specific confusing gesture pairs.
  - If possible, compare feature sets as well, such as raw time-window features vs handcrafted features.

- **Design the real-time demo carefully.**
  - Choose the window size and step size carefully.
  - You can choose to use temporal majority voting to smooth unstable predictions.
  - You may also try model majority voting, where several models predict from the same input and vote on the final label.
  - Keep the model small enough to run in real time.
  - Make the demo GUI clear enough for the audience to understand the current prediction (You can make it fancy!).
  - Measure responsiveness: how long does the system take to produce the correct prediction after the gesture is performed?
  - Once you have a real-time demo and it is working well, record a video as the backup.

- **Consider optional extensions (This makes your system more robust and cooler).**
  - Missing modality during inference: if one sensor, such as mmWave, disconnects, can the system still make a reasonable prediction?
  - Data augmentation: for time-series data, try lightweight augmentation such as Gaussian noise, small time shifts, scaling, or time warping.
  - You can even try [Test Time Augmentation](https://albumentations.ai/docs/4-advanced-guides/test-time-augmentation/)
  - Varying input window size: I noticed most groups use one fixed window size, but some gestures are shorter and some are longer. So an natural extension is to make the system support varying input window input.
  - Robustness to environment changes: if the user moves around or the surrounding multipath environment changes, does the model retain similar accuracy?
  - False-positive testing: if the user performs an unlisted motion, such as walking or stretching, does the system incorrectly trigger a known gesture?

We will continue updating this list, so check this page again later (Latest Update: 10:50 AM, July 27).
