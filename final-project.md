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

Your final poster should at least show:

1. the sensing setup;
2. the gestures you attempted;
3. the dataset size;
4. your model pipeline;
5. your evaluation results;
6. one limitation and one future improvement.


## Practical Tips

- Keep the physical setup repeatable.
- Balance the number of trials per gesture.
- Use clear file and folder names.
- Save raw data and processed features separately.
- Run a small training test early, even with messy data.
- Add gestures only after your evaluation code works.
- Prefer a reliable 6-gesture system over an unreliable 15-gesture system with no analysis.
- The IMU sensor might not survey 100Hz update rate if you connect a lot of USB devices. Once finish a data collection, check if the IMU sensor data make senses. If not, check the IMU lab page and code to update it to for example, 50 Hz / 25 Hz.
- When you need to develop multiple sensor system, first do smoke test and make sure everything is good.
- Some groups might want to use 3 UWB nodes. The correct way is to use 1 Controller and 2 Controllee. The code from the UWB lab only supports 1 Controller and 1 Controllee. You should check the device documentations and tune the command lines and parameters (such as slot number, etc.)
- Again, different UWB boards can interfere, use the channel and preamble code assigned in the google sheet for each group
- Similarily, make sure your mmWave radar direction is not directly pointing to other groups' radar
- If the hardware is not responding, try replugging them first, then check the cable connections (whether the radar connections are correct on both sides, i.e., the radar board and the convertor), whether the UWB board is flashing green light, etc.

We will keep updating the list. Come back and check later : )
