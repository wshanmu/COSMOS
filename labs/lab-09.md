---
layout: page
title: "Lab 9: UWB Ranging and Bimanual Activity Recognition"
description: IoT lab on Qorvo DWM3001CDK UWB ranging, 50 Hz FiRa TWR configuration, KDE analysis, and two-wrist activity classification.
parent: Labs
nav_order: 9
---

# Lab 9: UWB Ranging and Bimanual Activity Recognition **IoT**{: .label .label-iot }

Date: July 17  
Time: 1:00-5:00 PM  
TA: Shanmu Wang  
Hardware: Qorvo DWM3001CDK boards  

## Goals

By the end of this lab, each group should be able to:

- install and run Qorvo's UWB command-line tools;
- run FiRa two-way ranging on a pair of DWM3001CDK boards;
- explain how FiRa ranging uses timed message exchanges and scheduling slots;
- configure the ranging update rate to about 50 Hz;
- collect and analyze ranging measurements at fixed distances;
- complete a KDE plot script for ranging logs;
- collect a small two-wrist activity dataset from the inter-hand distance signal;
- train and evaluate a classifier for bimanual activity recognition.

## Materials

Download the UWB lab materials from the password-protected Box folder:

[Open lab materials on Box](https://ucla.app.box.com/folder/397751133396){: .btn .btn-primary }

Ask Eric for the password.

Download:

```text
uwb-qorvo-tools.zip
UWB_lab/
```

Keep the two folders next to each other after unzipping:

```text
SDK/Tools/
  uwb-qorvo-tools/
  UWB_lab/
```

Important references:

- `UWB-Qorvo-Tools-guide.pdf`, especially Section 1.3, **Quick Start**
- `uwb-qorvo-tools/scripts/fira/run_fira_twr/STUDENT_LAB_OPERATION.md`
- [Qorvo forum explanation of FiRa timing parameters](https://forum.qorvo.com/t/getting-fastest-distance-reading-on-dmw3001cdk/19322/2)

## 1:00-1:25 PM - Install UWB Qorvo Tools

The Qorvo manual's Section 1.3 shows how to install `uwb-qorvo-tools`. The manual creates a new Python virtual environment, but for this course you should reuse the Lab 1 Conda environment.

Open a terminal and go to the unzipped tool folder:

```bash
cd path/to/SDK/Tools/uwb-qorvo-tools
conda activate cosmos-ds
python -m pip install --upgrade pip
python -m pip install .
```

Check that the tools are available:

```bash
uqt_info
uqt_ls
```

If `uqt_info` is not found, keep using direct script paths such as:

```bash
python scripts/device/get_device_info/get_device_info.py -h
```

Then install any UWB lab code dependencies if the TA-provided `UWB_lab` folder includes a `requirements.txt`:

```bash
cd ../UWB_lab
python -m pip install -r requirements.txt
```

If there is no `requirements.txt`, continue and install missing packages only if Python reports an import error.

## 1:25-1:45 PM - Form Groups and Identify Ports

Form a group of 3.

Each group receives two DWM3001CDK boards:

- one **controller** board;
- one **controlee** board.

Each group must use its group ID as the FiRa preamble index. This helps reduce interference when multiple groups run UWB experiments in the same room.

Example:

```text
Group 3 -> --preamble-idx 3
Group 8 -> --preamble-idx 8
```

Both boards in the same group must use the same preamble index. Do not use another group's preamble index.

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

Set these variables in each terminal. Replace the examples with your real ports and group ID.

macOS/Linux:

```bash
export CONTROLLER_PORT=/dev/cu.usbmodemXXXXXXXX
export CONTROLEE_PORT=/dev/cu.usbmodemYYYYYYYY
export GROUP_ID=3
```

Windows PowerShell:

```powershell
$env:CONTROLLER_PORT="COM5"
$env:CONTROLEE_PORT="COM6"
$env:GROUP_ID="3"
```

Check both boards:

```bash
cd path/to/SDK/Tools/uwb-qorvo-tools
conda activate cosmos-ds
python scripts/device/get_device_info/get_device_info.py -p $CONTROLLER_PORT
python scripts/device/get_device_info/get_device_info.py -p $CONTROLEE_PORT
```

Both should report `status: Ok`.

## 1:45-2:10 PM - Run the Default FiRa TWR Demo

First, run the default FiRa TWR demo exactly once to confirm the boards and ports work.

Open two terminals. In both terminals:

```bash
cd path/to/SDK/Tools/uwb-qorvo-tools
conda activate cosmos-ds
```

### Terminal 1: Controlee

Start the controlee first:

```bash
python scripts/fira/run_fira_twr/run_fira_twr.py \
  -p $CONTROLEE_PORT \
  --controlee \
  --preamble-idx $GROUP_ID \
  --aoa-report all-disabled \
  -t 35
```

### Terminal 2: Controller

Start the controller second:

```bash
python scripts/fira/run_fira_twr/run_fira_twr.py \
  -p $CONTROLLER_PORT \
  --preamble-idx $GROUP_ID \
  --aoa-report all-disabled \
  -t 30
```

Successful output should include:

```text
status: Ok (0x0)
distance: <value> cm
```

Checkpoint: show the TA one live successful distance output.

## 2:10-2:35 PM - Understand and Configure 50 Hz Ranging

The default demo is intentionally conservative. The Qorvo forum explanation describes three timing concepts:

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

For this lab, configure about 50 Hz:

```text
target update rate = 50 Hz
ranging interval = 1000 / 50 = 20 ms
slots per ranging round = 6
minimum slot time = 6 * 2 ms = 12 ms
20 ms >= 12 ms, so this configuration is valid
```

Set these variables in both terminals:

```bash
export SLOT_SPAN=2400
export SLOTS_PER_RR=6
export RANGING_SPAN=20
```

Windows PowerShell:

```powershell
$env:SLOT_SPAN="2400"
$env:SLOTS_PER_RR="6"
$env:RANGING_SPAN="20"
```

Run the 50 Hz version.

Controlee:

```bash
python scripts/fira/run_fira_twr/run_fira_twr.py \
  -p $CONTROLEE_PORT \
  --controlee \
  --preamble-idx $GROUP_ID \
  --aoa-report all-disabled \
  --slot-span $SLOT_SPAN \
  --slots-per-rr $SLOTS_PER_RR \
  --ranging-span $RANGING_SPAN \
  -t 35
```

Controller:

```bash
python scripts/fira/run_fira_twr/run_fira_twr.py \
  -p $CONTROLLER_PORT \
  --preamble-idx $GROUP_ID \
  --aoa-report all-disabled \
  --slot-span $SLOT_SPAN \
  --slots-per-rr $SLOTS_PER_RR \
  --ranging-span $RANGING_SPAN \
  -t 30
```

The controller output should show:

```text
ranging interval: 20 ms
```

Question: why does reducing `--ranging-span` increase the update rate, and why must it still be larger than the required slot time?

## 2:35-3:10 PM - Collect 30-Second Ranging Logs

Now collect fixed-distance logs with the higher-level `UWB_lab` wrapper. This wrapper starts both sides, saves terminal logs, parses ranging samples, and writes session folders.

Go to the UWB lab codebase:

```bash
cd path/to/SDK/Tools/UWB_lab
conda activate cosmos-ds
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
  --duration 30 \
  --fps 50 \
  --session-name group_${GROUP_ID}_far_250cm
```

Record:

| Session | True distance | Session folder | Mean distance | Notes |
| --- | --- | --- | --- | --- |
| Near |  |  |  |  |
| Far |  |  |  |  |

## 3:10-3:35 PM - Finish the KDE Plot Script

Open:

```text
UWB_lab/analyze_ranging_results.py
```

Finish the TODOs for plotting the KDE distribution of the ranging logs. Your plot should show the distribution of successful distance measurements after basic filtering.

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

## 3:35-4:40 PM - Bimanual Activity Recognition

Now use UWB ranging for sensing.

Research question:

> Can we recognize two-handed activities using only the distance between a person's two wrists?

Place one DWM3001CDK on each wrist or forearm. The classifier will use the inter-hand distance signal over time.

Choose at least 4 activity classes. Possible classes:

- `clapping`
- `open_close_arms`
- `folding_towel`
- `passing_object`
- `stirring`
- `putting_on_glove`
- `tying_knot`
- `resting`

Recommended starting set if time is short:

```text
clapping,open_close_arms,passing_object,resting
```

### Collect Data

Each group member should collect data as a separate `collector`. This makes leave-one-user-out evaluation possible.

Example for one collector:

```bash
python collect_dataset.py \
  --controller-port $CONTROLLER_PORT \
  --controlee-port $CONTROLEE_PORT \
  --group-id $GROUP_ID \
  --collector student01 \
  --gesture clapping,open_close_arms,passing_object,resting \
  --input-type range \
  --trials 8 \
  --trial-duration 5 \
  --pause 3 \
  --fps 50
```

Repeat with `--collector student02` and `--collector student03` when other group members perform the activities.

During collection:

1. Wait for the prepare cue.
2. Start the activity only when the capture cue appears.
3. Keep the activity consistent across trials.
4. Reject bad trials by answering `N` when the script asks whether to keep the trial.
5. Keep a trial only if it has enough successful range samples.

### Feature Design TODOs

The codebase provides a starting feature extractor for range windows, but you should inspect and improve it.

Look at:

```text
UWB_lab/uwb_lab_common.py
```

Find the range feature extraction logic and consider features such as:

- mean, standard deviation, min, max, and median distance;
- interquartile range;
- first and last distance;
- total change over the trial;
- mean and max step size;
- slope over time;
- resampled signal shape;
- periodicity or peak-count features for repeated actions such as clapping.

Your group should finish the TODOs added by the TA and add at least one feature that you believe helps distinguish the chosen activities.

### Train and Evaluate

Train a Random Forest classifier:

```bash
python train.py datasets/gesture_dataset_YYYYMMDD_HHMMSS \
  --input-type range \
  --side controller
```

If your group collected separate datasets for multiple collectors, combine them:

```bash
python combine_datasets.py \
  datasets/gesture_dataset_student01 \
  datasets/gesture_dataset_student02 \
  datasets/gesture_dataset_student03 \
  --input-type range \
  --output datasets/combined_bimanual_range
```

Train with a random split:

```bash
python train.py datasets/combined_bimanual_range \
  --input-type range \
  --side controller
```

Then run leave-one-user-out evaluation. For example, hold out `student03`:

```bash
python train.py datasets/combined_bimanual_range \
  --input-type range \
  --side controller \
  --test-collector student03
```

Required evaluation:

1. Random train/test accuracy.
2. Confusion matrix.
3. Leave-one-user-out accuracy for at least one held-out collector.
4. A short discussion of which activities are confused and why.

Question: does the model still work when the test user was not in the training set?

## 4:40-5:00 PM - Checkoff and Submission

Show the TA:

1. Default `run_fira_twr.py` ranging output.
2. 50 Hz ranging output showing a 20 ms ranging interval.
3. Near-distance and far-distance 30-second session folders.
4. KDE plots for both ranging sessions.
5. Your selected bimanual activity classes.
6. At least one collected activity dataset.
7. A trained classifier summary and confusion matrix.
8. Leave-one-user-out evaluation result, if your group collected data from multiple users.

Submit or paste into your group notes:

1. Group ID and preamble index.
2. Controller and controlee port names.
3. Near and far ground-truth distances.
4. KDE plots and summary numbers.
5. Activity labels and number of accepted trials per label.
6. Feature changes you made or TODOs you completed.
7. Random split accuracy.
8. Leave-one-user-out accuracy.
9. One paragraph answering the research question.

## Troubleshooting

| Problem | Likely cause | Fix |
| --- | --- | --- |
| `get_device_info.py` does not see the board | Wrong port, cable, or board busy | Check USB cable, close other terminals, replug the board, and rerun the command. |
| Default TWR has repeated timeouts | Ports swapped, wrong preamble, or interference | Start controlee first, confirm both boards use the same `--preamble-idx`, and move boards closer. |
| Other groups interfere | Shared preamble or nearby UWB traffic | Use your assigned group ID as the preamble index and do not reuse another group's code. |
| 50 Hz run fails | Timing parameters invalid or host cannot keep up | Use `--slot-span 2400`, `--slots-per-rr 6`, `--ranging-span 20`; reduce FPS if needed. |
| KDE script finds no samples | Wrong session path or no `Ok` measurements | Check the session folder and inspect `controller/ranging_samples.csv` or terminal logs. |
| Classifier accuracy is too good to be trusted | Data leakage across trials or users | Evaluate with `--test-collector` and avoid splitting windows from the same trial into both train and test. |
| Leave-one-user-out accuracy is low | User-specific motion patterns | Collect more users, normalize features, or add features that describe motion shape rather than absolute wrist distance only. |

## Core Takeaway

UWB ranging is not only useful for localization. A time series of inter-device distance can become a sensing signal. In this lab, the same FiRa TWR measurements are first evaluated as distance estimates and then reused as features for bimanual activity classification.
