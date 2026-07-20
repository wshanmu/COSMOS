---
layout: page
title: "Lab 9: UWB Ranging and Bimanual Activity Recognition"
description: IoT lab on Qorvo DWM3001CDK UWB ranging, FiRa TWR timing, KDE analysis, and two-wrist activity classification.
parent: Labs
nav_order: 9
---

# Lab 9: UWB Ranging and Bimanual Activity Recognition **IoT**{: .label .label-iot }

Date: July 17  
Lab window: 1:00-5:00 PM  
TA: Shanmu Wang  
Hardware: Qorvo DWM3001CDK boards  

## Goals

By the end of this lab, each group should be able to:

- fork, clone, and run the UWB lab repository;
- run FiRa two-way ranging on a pair of DWM3001CDK boards;
- explain how UWB ranging estimates distance from time-of-flight;
- explain why preamble code and channel assignment reduce interference between groups;
- configure the ranging update rate to about 50 Hz;
- collect and analyze ranging measurements at fixed distances;
- complete a KDE plot script for ranging logs;
- collect a small two-wrist activity dataset from the inter-hand distance signal;
- train, evaluate, and improve a classifier for bimanual activity recognition;
- test a trained model in real time with majority voting.

## Materials and Repository

Use the course UWB lab repository:

[Open UWB lab repository](https://github.com/wshanmu/UWB_lab){: .btn .btn-primary }

Each group member should fork [github.com/wshanmu/UWB_lab](https://github.com/wshanmu/UWB_lab), then clone their own fork so they can edit code and push changes.

macOS/Linux:

```bash
git clone https://github.com/YOUR_GITHUB_USERNAME/UWB_lab.git
cd UWB_lab
conda activate py39
```

Windows PowerShell:

```powershell
git clone https://github.com/YOUR_GITHUB_USERNAME/UWB_lab.git
cd UWB_lab
conda activate py39
```

After cloning, follow the `README.md` in the repository. The repository already contains the `uwb-qorvo-tools` subfolder, so you do not need to install the Qorvo tools as a separate Python package.

Important references:

- `UWB-Qorvo-Tools-guide.pdf`, especially Section 1.3, **Quick Start**
- `uwb-qorvo-tools/scripts/fira/run_fira_twr/STUDENT_LAB_OPERATION.md`
- [Qorvo forum explanation of FiRa timing parameters](https://forum.qorvo.com/t/getting-fastest-distance-reading-on-dmw3001cdk/19322/2)

## Step Index

1. Set up the repository and group assignment.
2. Identify the serial ports for the two boards.
3. Run the default FiRa DS-TWR ranging demo.
4. Configure the channel, preamble code, and 50 Hz ranging parameters.
5. Collect fixed-distance ranging logs.
6. Finish the KDE visualization.
7. Collect bimanual activity data.
8. Design features and train classifiers.
9. Evaluate with random split and leave-one-collector-out split.
10. Run real-time classification and implement majority voting.

## Section 1: Setup and Group Assignment

### Step 1: Fork and Clone the Repository

Fork the UWB lab repository to your own GitHub account, then clone your fork. Do not clone another student's fork.

```bash
git clone https://github.com/YOUR_GITHUB_USERNAME/UWB_lab.git
cd UWB_lab
conda activate py39
```

Create a new Conda Python Environment:

```bash
conda create -n py39 python=3.9
conda activate py39
```

```bash
cd uwb-qorvo-tools
pip install -e .
pip install -e lib/uqt-utils/
pip install -e lib/uwb-uci/
```

The Qorvo manual explains how to install the Qorvo tools as a standalone package. For this lab, use the provided course repository instead. The scripts in this repository set the needed Qorvo paths when they launch subprocesses.

### Step 2: Form a Group and Record the Assignment

Form a group of 3 in the class sheet:

[Open UWB group sheet](https://docs.google.com/spreadsheets/d/1xEM-HSpBDMrXdBSz2BphMjZDY_hA6qhXVAFCRKVvYuw/edit?usp=sharing){: .btn .btn-primary }

Each group receives two DWM3001CDK boards:

- one **controller** board;
- one **controlee** board.

There are eight groups total. Each group must use the preamble code and UWB channel assigned in the sheet. The preamble code will be one of `9, 10, 11, 12`, and the channel will be `5` or `9`.

Example sheet assignment:

```text
Group 1 -> GROUP_ID=1, PREAMBLE_CODE=9, UWB_CHANNEL=5
```

Both boards in the same group must use the same preamble code and channel. Do not use another group's assignment.

### Step 3: Identify Serial Ports

Connect both boards and find their serial ports.

macOS:

```bash
ls /dev/cu.usbmodem*
```

Linux:

```bash
ls /dev/serial/by-id/*
```

Windows:

```text
Device Manager -> Ports (COM & LPT)
```

Set these variables in each terminal. Replace the examples with your real ports, group ID, preamble code, and channel from the sheet.

macOS/Linux:

```bash
export CONTROLLER_PORT=/dev/cu.usbmodemXXXXXXXX
export CONTROLEE_PORT=/dev/cu.usbmodemYYYYYYYY
export GROUP_ID=1
export PREAMBLE_CODE=9
export UWB_CHANNEL=5
```

Windows PowerShell:

```powershell
$env:CONTROLLER_PORT="COM5"
$env:CONTROLEE_PORT="COM6"
$env:GROUP_ID="1"
$env:PREAMBLE_CODE="9"
$env:UWB_CHANNEL="5"
```

### Step 4: Check Both Boards

From the cloned repository root:

```bash
cd uwb-qorvo-tools
conda activate py39
python scripts/device/get_device_info/get_device_info.py -p $CONTROLLER_PORT
python scripts/device/get_device_info/get_device_info.py -p $CONTROLEE_PORT
```

Both should report `status: Ok`.

## Section 2: UWB Ranging Basics and Default DS-TWR

UWB ranging estimates distance from time-of-flight:

```text
distance = time of flight * speed of light
```

The hard part is that the two devices do not share exactly the same clock. A packet timestamp from one board cannot be directly compared with a timestamp from the other board. FiRa double-sided two-way ranging, or DS-TWR, solves this by exchanging several scheduled messages and using the send and receive timestamps together.

### Step 5: Run the Default FiRa TWR Demo

First, run the default FiRa TWR demo exactly once to confirm the boards and ports work.

Open two terminals. In both terminals, enter the Qorvo tool folder from the repository root:

```bash
cd uwb-qorvo-tools
conda activate py39
```

Start the controlee first.

**Terminal 1: Controlee**

```bash
python scripts/fira/run_fira_twr/run_fira_twr.py \
  -p $CONTROLEE_PORT \
  --controlee \
  --channel $UWB_CHANNEL \
  --preamble-idx $PREAMBLE_CODE \
  --aoa-report all-disabled \
  -t 35
```

Start the controller second.

**Terminal 2: Controller**

```bash
python scripts/fira/run_fira_twr/run_fira_twr.py \
  -p $CONTROLLER_PORT \
  --channel $UWB_CHANNEL \
  --preamble-idx $PREAMBLE_CODE \
  --aoa-report all-disabled \
  -t 30
```

Successful output should include:

```text
status: Ok (0x0)
distance: <value> cm
```

Checkpoint: show the TA one live successful distance output.

## Section 3: Preamble Code, Channel, and 50 Hz Ranging

The receiver detects a UWB packet by correlating the incoming signal with the packet preamble. If nearby groups reuse the same channel and preamble code, they can interfere with each other. Use the `{channel, preamble code}` pair assigned to your group in the sheet.

### Step 6: Understand the Timing Parameters

The Qorvo forum explanation describes three timing concepts:

| Concept | Meaning in this lab |
| --- | --- |
| Block duration / ranging interval | Time budget for one complete ranging sequence. In the script, this is controlled by `--ranging-span`, in milliseconds. |
| Slot duration | Time allocated for one message to be sent or received. In the script, `--slot-span 2400` corresponds to about 2 ms. |
| Slots per ranging round | Number of slots reserved for one ranging round. In the script, this is `--slots-per-rr`. |

The default FiRa DS-TWR deferred mode uses scheduled messages such as:

| Message | Meaning |
| --- | --- |
| `CM` | Control Message |
| `RI` | Ranging Initiation |
| `RR` | Ranging Response |
| `RF` | Ranging Final |
| `MR` | Measurement Report |
| `RRR` | Ranging Result Report |

For one controller and one controlee, we do not need the large default slot reservation.

Default settings:

```text
--slot-span 2400
--slots-per-rr 25
--ranging-span 200
```

Interpretation:

```text
slot duration ~= 2 ms
25 slots * 2 ms = 50 ms of reserved slot time
ranging interval = 200 ms
update rate = 1000 / 200 = 5 Hz
```

### Step 7: Compute the 50 Hz Configuration

For this lab, configure about 50 Hz, but do the calculation yourself:

```text
target update rate = 50 Hz
slot duration (ms) = slot-span / 1200
minimum ranging-span = slot duration * slots-per-rr
target ranging-span = 1000 / target update rate
slot-span = 2400
slots per ranging round = 6
```

TODO: compute a valid `RANGING_SPAN`. It must be large enough for all slots in the ranging round, and it should give about 50 Hz.

Set these variables in both terminals after you fill in the TODO:

macOS/Linux:

```bash
export SLOT_SPAN=2400
export SLOTS_PER_RR=6
export RANGING_SPAN=TODO_RANGING_SPAN
```

Windows PowerShell:

```powershell
$env:SLOT_SPAN="2400"
$env:SLOTS_PER_RR="6"
$env:RANGING_SPAN="TODO_RANGING_SPAN"
```

### Step 8: Run the 50 Hz Version

Run the same demo again with your computed timing parameters.

**Controlee**

```bash
python scripts/fira/run_fira_twr/run_fira_twr.py \
  -p $CONTROLEE_PORT \
  --controlee \
  --channel $UWB_CHANNEL \
  --preamble-idx $PREAMBLE_CODE \
  --aoa-report all-disabled \
  --slot-span $SLOT_SPAN \
  --slots-per-rr $SLOTS_PER_RR \
  --ranging-span $RANGING_SPAN \
  -t 35
```

**Controller**

```bash
python scripts/fira/run_fira_twr/run_fira_twr.py \
  -p $CONTROLLER_PORT \
  --channel $UWB_CHANNEL \
  --preamble-idx $PREAMBLE_CODE \
  --aoa-report all-disabled \
  --slot-span $SLOT_SPAN \
  --slots-per-rr $SLOTS_PER_RR \
  --ranging-span $RANGING_SPAN \
  -t 30
```

The controller output should show:

```text
ranging interval: <your computed RANGING_SPAN> ms
```

Question: why does reducing `--ranging-span` increase the update rate, and why must it still be larger than the required slot time?

## Section 4: Fixed-Distance Ranging Experiment and KDE

Now collect controlled ranging logs. The goal is to see the measurement distribution, not just one distance value.

### Step 9: Collect Two 30-Second Ranging Logs

Use the higher-level wrapper in the cloned `UWB_lab` repository. This wrapper starts both sides, saves terminal logs, parses ranging samples, and writes session folders.

Open a terminal in the `UWB_lab` repository root, which is the folder containing `ranging_experiment_wrapper.py`, and activate the environment:

```bash
conda activate py39
```

Measure two distances:

| Trial | True distance | Duration | Notes |
| --- | --- | --- | --- |
| Near | about 50 cm | 30 s | Keep boards stable. |
| Far | as far as possible, up to 2.5 m | 30 s | Keep line of sight if possible. |

Near-distance command:

```bash
python ranging_experiment_wrapper.py \
  --controller-port $CONTROLLER_PORT \
  --controlee-port $CONTROLEE_PORT \
  --group-id $GROUP_ID \
  --preamble-code $PREAMBLE_CODE \
  --channel $UWB_CHANNEL \
  --duration 30 \
  --fps 50 \
  --session-name group_${GROUP_ID}_near_50cm
```

Far-distance command:

```bash
python ranging_experiment_wrapper.py \
  --controller-port $CONTROLLER_PORT \
  --controlee-port $CONTROLEE_PORT \
  --group-id $GROUP_ID \
  --preamble-code $PREAMBLE_CODE \
  --channel $UWB_CHANNEL \
  --duration 30 \
  --fps 50 \
  --session-name group_${GROUP_ID}_far_250cm
```

Record:

| Session | True distance | Session folder | Mean distance | Notes |
| --- | --- | --- | --- | --- |
| Near |  |  |  |  |
| Far |  |  |  |  |

### Step 10: Finish the KDE Plot Script

Open:

```text
analyze_ranging_results.py
```

Finish the TODOs for visualizing the KDE distribution of the ranging logs. Your plot should show the distribution of successful distance measurements after basic filtering.

You may use either approach:

- compute the KDE density yourself, then call Matplotlib to draw the curve;
- call Seaborn's KDE plotting function directly on the filtered distance values.

Useful API references:

- [Matplotlib `pyplot.plot`](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.plot.html)
- [Matplotlib `pyplot.hist`](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.hist.html)
- [Seaborn `kdeplot`](https://seaborn.pydata.org/generated/seaborn.kdeplot.html)

Run the analysis on both sessions:

```bash
python analyze_ranging_results.py sessions/group_${GROUP_ID}_near_50cm --side controller
python analyze_ranging_results.py sessions/group_${GROUP_ID}_far_250cm --side controller
```

If your session folders were saved somewhere else, replace the paths with the actual session paths.

Paste the following into your group notes or slides:

1. KDE plot for the near-distance trial.
2. KDE plot for the far-distance trial.
3. The mean and median distance from the summary JSON.
4. One sentence comparing the two distributions.

Questions:

1. Which distance had a tighter distribution?
2. Did the measured distance show a clear bias relative to the tape-measured distance?
3. Did any outliers appear? If so, what might have caused them?

## Section 5: Bimanual Activity Recognition Dataset

Now use UWB ranging as a sensing signal.

Research question:

> Can we recognize two-handed activities using only the distance between a person's two wrists?

Place one DWM3001CDK on each wrist or forearm. The classifier will use the inter-hand distance signal over time. This signal is only one-dimensional, so the model must rely on motion patterns such as repeated closing, opening, or changes in rhythm.

### Step 11: Choose Activity Classes

Choose at least 4 activity classes. Possible classes:

- `clapping`
- `T_arms`
- `boxing`
- `clock_circle`
- `resting`

Recommended starting set if time is short:

```text
clapping,T_arms,boxing,resting
```

### Step 12: Collect Data From Multiple Collectors

Each group member should collect data as a separate `collector`. This makes leave-one-user-out evaluation possible.

Example for one collector:

```bash
python collect_dataset.py \
  --controller-port $CONTROLLER_PORT \
  --controlee-port $CONTROLEE_PORT \
  --group-id $GROUP_ID \
  --preamble-code $PREAMBLE_CODE \
  --channel $UWB_CHANNEL \
  --collector student01 \
  --gesture clapping,T_arms,boxing,resting \
  --trials 10 \
  --trial-duration 3 \
  --pause 2 \
  --fps 50
```

Repeat with `--collector student02` and `--collector student03` when other group members perform the activities.

During collection:

1. Wait for the prepare cue.
2. Start the activity only when the capture cue appears.
3. Keep the activity consistent across trials.
4. Reject bad trials by answering `N` when the script asks whether to keep the trial.
5. Keep a trial only if it has enough successful range samples.

### Step 13: Combine Datasets

If your group collected separate datasets for multiple collectors, combine them:

```bash
python combine_datasets.py \
  datasets/gesture_dataset_student01 \
  datasets/gesture_dataset_student02 \
  datasets/gesture_dataset_student03 \
  --output datasets/combined_bimanual_range
```

## Section 6: Feature Engineering and Model Choice

The default model and default feature set are intentionally basic. Your task is to improve the sensing pipeline by deciding what information from the distance signal should matter.

### Step 14: Inspect the Baseline Features

Look at:

```text
uwb_lab_common.py
```

The baseline extractor already provides:

- mean, standard deviation, min, max, and median distance;
- interquartile range;
- first and last distance;
- total change over the trial;
- mean and max step size;
- slope over time;
- resampled signal shape.

### Step 15: Add Lightweight Proposal Features

The proposal extractor currently appends placeholder values. Replace those placeholders with lightweight features that you believe help distinguish your chosen activities.

Useful directions include:

- fraction of time the wrists are close together;
- beginning-to-end distance change;
- total movement amount, such as the sum of absolute distance differences;
- number of direction changes;
- number, depth, or spacing of valleys when the hands come together;
- simple trend or segment-level features.

Avoid expensive features that would slow down real-time evaluation.

### Step 16: Train the Starter Models

Train a baseline KNN classifier:

```bash
python train.py datasets/combined_bimanual_range \
  --side controller \
  --feature-set baseline \
  --classifier knn \
  --knn-neighbors 5
```

Compare with the starter linear SVM:

```bash
python train.py datasets/combined_bimanual_range \
  --side controller \
  --feature-set baseline \
  --classifier svm_linear \
  --svm-c 1.0
```

Then compare baseline features with your proposal features:

```bash
python train.py datasets/combined_bimanual_range \
  --side controller \
  --feature-set proposal \
  --classifier knn
```

### Step 17: Add at Least One Stronger Classifier

Classifier TODO: add at least one additional classifier by editing `build_classifier()` in `train.py`.

Good starting references:

- [scikit-learn supervised learning overview](https://scikit-learn.org/stable/supervised_learning.html)
- [RandomForestClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html)
- [DecisionTreeClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeClassifier.html)
- [Support Vector Machines with different kernels](https://scikit-learn.org/stable/modules/svm.html)

Suggested models to try:

- SVM with RBF kernel;
- SVM with polynomial kernel;
- decision tree;
- random forest;
- KNN with different values of `k`.

## Section 7: Model Evaluation

Accuracy from a random train/test split is useful, but it may overestimate performance if similar trials from the same person appear in both train and test. You must also test whether the model generalizes to a collector who was not used for training.

### Step 18: Evaluate a Random Split

Train with a random split:

```bash
python train.py datasets/combined_bimanual_range \
  --side controller \
  --classifier knn
```

Record the accuracy and confusion matrix.

### Step 19: Evaluate Leave-One-Collector-Out

For example, hold out `student03`:

```bash
python train.py datasets/combined_bimanual_range \
  --side controller \
  --test-collector student03
```

Repeat with different held-out collectors if you have time.

Required evaluation:

1. Random train/test accuracy.
2. Confusion matrix.
3. Leave-one-collector-out accuracy for at least one held-out collector.
4. A short discussion of which activities are confused and why.

Question: does the model still work when the test collector was not in the training set?

## Section 8: Real-Time Classification and Majority Voting

The real-time evaluator makes predictions from sliding windows. For example, with a 3-second window and `--step-seconds 0.5`, the classifier predicts from overlapping windows:

```text
t = 3.0 s -> classify 0.0 to 3.0 s
t = 3.5 s -> classify 0.5 to 3.5 s
t = 4.0 s -> classify 1.0 to 4.0 s
```

A smaller step gives more frequent updates, but also requires more computation.

### Step 20: Run Raw Real-Time Predictions

After training a model, run the real-time evaluator with raw predictions:

```bash
python eval_realtime.py \
  --model datasets/combined_bimanual_range/models/CLASSIFIER_range_YYYYMMDD_HHMMSS.joblib \
  --controller-port $CONTROLLER_PORT \
  --controlee-port $CONTROLEE_PORT \
  --group-id $GROUP_ID \
  --preamble-code $PREAMBLE_CODE \
  --channel $UWB_CHANNEL \
  --duration 60 \
  --step-seconds 0.5 \
  --vote-window 1 \
  --visualize
```

The evaluator stores the model output in `realtime_predictions.csv`. With `--vote-window 1`, the display shows the raw prediction from each sliding window.

### Step 21: Implement Majority Voting

Individual predictions may be unstable. Majority voting keeps the most recent predictions and outputs the most frequent class.

Example with a vote window of 5:

```text
raw predictions:   Clap, Clap, Clap, Clap, Clap, Box, Clap, Clap
voted predictions: ----, ----, ----, ----, Clap, Clap, Clap, Clap
```

TODO: implement `majority_vote()` in `eval_realtime.py`. After that, try:

```bash
python eval_realtime.py \
  --model datasets/combined_bimanual_range/models/CLASSIFIER_range_YYYYMMDD_HHMMSS.joblib \
  --controller-port $CONTROLLER_PORT \
  --controlee-port $CONTROLEE_PORT \
  --group-id $GROUP_ID \
  --preamble-code $PREAMBLE_CODE \
  --channel $UWB_CHANNEL \
  --duration 60 \
  --step-seconds 0.5 \
  --vote-window 5 \
  --visualize
```

Compare raw predictions and majority-voted predictions. What changes faster: responsiveness or stability?

### Step 22: Optional Model Voting

If time allows, try majority voting across multiple trained models. For example, compare the output of KNN, linear SVM, and a tree-based model on the same real-time window, then output the class chosen by most models.

## Section 9: Optional Extensions

Use these only after the required lab tasks are working.

1. Add data augmentation, such as Gaussian noise, randomized starting offset, or speed variation.
2. Try synchronized multi-sensor collection, such as UWB plus IMU.
3. Compare models when sensors have different sampling rates.
4. Watch the related UWB activity-recognition demo: [recent work video](https://www.youtube.com/watch?v=6PrZR7REcQg).

## Checkoff and Submission

Show the TA:

1. Default `run_fira_twr.py` ranging output.
2. 50 Hz ranging output showing the ranging interval you computed.
3. Near-distance and far-distance 30-second session folders.
4. KDE plots for both ranging sessions.
5. Your selected bimanual activity classes.
6. At least one collected activity dataset.
7. A trained classifier summary and confusion matrix.
8. Leave-one-collector-out evaluation result, if your group collected data from multiple users.
9. Raw real-time predictions and, after you implement it, majority-voted predictions.

Submit or paste into your group notes:

1. Group ID, preamble code, and UWB channel from the sheet.
2. Controller and controlee port names.
3. Near and far ground-truth distances.
4. KDE plots and summary numbers.
5. Activity labels and number of accepted trials per label.
6. Feature changes you made or TODOs you completed.
7. Random split accuracy.
8. Leave-one-collector-out accuracy.
9. Real-time voting behavior, if completed.
10. One paragraph answering the research question.

## Troubleshooting

| Problem | Likely cause | Fix |
| --- | --- | --- |
| `get_device_info.py` does not see the board | Wrong port, cable, or board busy | Check USB cable, close other terminals, replug the board, and rerun the command. |
| Default TWR has repeated timeouts | Ports swapped, wrong preamble, or interference | Start controlee first, confirm both boards use the same `--preamble-idx`, and move boards closer. |
| Other groups interfere | Shared preamble code, shared channel, or nearby UWB traffic | Use the preamble code and channel assigned in the sheet; do not reuse another group's assignment. |
| 50 Hz run fails | Timing parameters invalid or host cannot keep up | Recheck the `RANGING_SPAN` calculation and confirm it is not shorter than the required slot time; reduce FPS if needed. |
| KDE script finds no samples | Wrong session path or no `Ok` measurements | Check the session folder and inspect `controller/ranging_samples.csv` or terminal logs. |
| Classifier accuracy is too good to be trusted | Data leakage across trials or users | Evaluate with `--test-collector` and avoid splitting windows from the same trial into both train and test. |
| Leave-one-collector-out accuracy is low | User-specific motion patterns | Collect more users, normalize features, or add features that describe motion shape rather than absolute wrist distance only. |

## Core Takeaway

UWB ranging is not only useful for localization. A time series of inter-device distance can become a sensing signal. In this lab, the same FiRa TWR measurements are first evaluated as distance estimates and then reused as features for bimanual activity classification.
