---
layout: page
title: "Lab 9: UWB Ranging and CIR Sensing"
description: IoT lab on UWB ranging, LoS/NLoS performance, and CIR visualization using Qorvo DWM3001CDK boards.
parent: Labs
nav_order: 9
---

# Lab 9: UWB Ranging and CIR Sensing **IoT**{: .label .label-iot }

Date: July 17  
TA: Shanmu Wang  
Hardware: Qorvo DWM3001CDK boards  
Total time: About 4 hours

## Goals

By the end of this lab, each group should be able to:

- explain how UWB time-of-flight ranging estimates distance;
- run a double-sided two-way ranging (DS-TWR) demo on DWM3001CDK boards;
- measure ranging accuracy and stability under line-of-sight (LoS) and non-line-of-sight (NLoS) conditions;
- compute bias, standard deviation, MAE, RMSE, P90 error, and outlier rate from ranging logs;
- visualize channel impulse response (CIR) data as a delay-domain profile and a waterfall plot;
- use background subtraction to highlight motion-induced CIR changes;
- collect a small UWB sensing dataset for a simple hand-movement classification task.

## Hardware

The DWM3001CDK is Qorvo's development kit for evaluating the DWM3001C UWB module. For this lab, the most relevant board/module facts are:

| Item | Value |
| --- | --- |
| UWB module | DWM3001C, based on Qorvo DW3110 |
| MCU | Nordic nRF52833 Cortex-M4 BLE SoC on the DWM3001C module |
| UWB channels | Channel 5 around 6.5 GHz and channel 9 around 8 GHz |
| Intended roles | TWR tag/anchor or TDoA tag/anchor |
| Debug/flashing | On-board J-Link over USB |
| Interfaces | USB connection to J-Link and DWM3001C USB interface |
| Typical positioning use | Real-time location systems, asset tracking, and location-aware sensor networks |

Official references:

- [DWM3001CDK product page](https://www.qorvo.com/products/p/DWM3001CDK)
- [DWM3001C product page](https://www.qorvo.com/products/p/DWM3001C)
- [DW3xxx & QM3xxx SDK download](https://www.qorvo.com/products/d/da008582)
- [UWB Product Certification Process in the USA](https://www.qorvo.com/products/d/da008185)
- [UWB Worldwide Regulations](https://www.qorvo.com/products/d/da008184)

## Recommended Group Roles

- Hardware lead: connects boards, checks USB/J-Link, labels initiator/responder.
- Firmware lead: downloads SDK, flashes prebuilt images, records serial ports.
- Experiment lead: places boards, measures ground-truth distance, manages LoS/NLoS scenes.
- Data lead: saves logs, plots metrics, and prepares final answers.

Rotate roles between Part A and Part B if time allows.

## Before Lab

TA will prepare:

- one pair of DWM3001CDK boards per group;
- USB cables for both boards;
- download link for the Qorvo SDK resources;
- download link for a serial terminal software;

Please make sure each group at least has:

- one lab computer with Windows OS for setting up the SDK and flashing the firmware;
- PuTTY on Windows, or access to the built-in `screen` command on macOS for reading serial output.

### Serial Terminal Setup

The DWM3001CDK prints status messages, range estimates, and debugging text over a USB serial connection. A serial terminal is the program on your laptop that opens this port and shows the text stream.

Conceptually, the data path is:

```text
DWM3001CDK firmware -> USB virtual serial port -> PuTTY/screen -> text logs on your computer
```

The serial port name depends on your operating system:

- Windows usually names ports `COM3`, `COM4`, etc.
- macOS usually names ports `/dev/tty.usbmodem...` or `/dev/tty.usbserial...`.

The most important setting is the **baud rate**, which must match the firmware. In this lab, start with `115200` unless the TA gives a different value. If the baud rate or COM port is wrong, you may see no output or unreadable characters.

<details markdown="1">
<summary>Windows: PuTTY serial terminal</summary>

1. Download PuTTY from the [official PuTTY download page](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). The 64-bit Windows MSI installer is the easiest option for most lab computers.
2. Connect the DWM3001CDK by USB.
3. Open **Device Manager** -> **Ports (COM & LPT)** and find the new COM port, such as `COM5`.
4. Open PuTTY.
5. Under **Connection type**, select **Serial**.
6. Set **Serial line** to your COM port, such as `COM5`.
7. Set **Speed** to `115200`.
8. Click **Open**.
9. If the terminal is blank, press Enter once or reset the board.
10. To save logs, open **Session** -> **Logging** in PuTTY before starting the experiment.

PuTTY's official documentation has a short section on [connecting to a local serial line](https://the.earth.li/~sgtatham/putty/latest/htmldoc/Chapter3.html#using-serial).
</details>

<details markdown="1">
<summary>macOS: Terminal and screen</summary>

1. Connect the board by USB.
2. Open **Terminal**.
3. List likely serial ports:

   ```bash
   ls /dev/tty.usb* /dev/tty.usbmodem* 2>/dev/null
   ```

4. Open the port with `screen`, replacing the device name with the one on your laptop:

   ```bash
   screen /dev/tty.usbmodemXXXX 115200
   ```

5. If the terminal is blank, press Enter once or reset the board.
6. To leave `screen`, press `Ctrl-A`, then `Ctrl-\`, then type `y`.
7. To save logs, copy text from Terminal or ask the TA for a Python logging script.

Apple's [Terminal User Guide](https://support.apple.com/guide/terminal/welcome/mac) is a useful reference for the macOS command-line environment.
</details>

Further reading:

- [UART overview](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter)
- [PuTTY official documentation: serial line mode](https://the.earth.li/~sgtatham/putty/latest/htmldoc/Chapter3.html#using-serial)
- [Apple Terminal User Guide](https://support.apple.com/guide/terminal/welcome/mac)


## Part A: DS-TWR Ranging

### 0:00-0:15 - UWB and DS-TWR Concept

UWB ranging estimates distance from propagation time:

`distance = speed of light * time of flight`

In practice, two devices do not share a perfectly synchronized clock. Double-sided two-way ranging reduces clock-offset error by exchanging multiple timestamped messages between an initiator and a responder.

![Symmetrical double-sided two-way ranging timing diagram](https://upload.wikimedia.org/wikipedia/commons/thumb/c/cf/Sds-twr.png/250px-Sds-twr.png)

Figure: a symmetrical double-sided two-way ranging timing exchange. Source: [Wikimedia Commons / SDS-TWR](https://en.wikipedia.org/wiki/Symmetrical_double-sided_two-way_ranging).

A useful mental model is:

1. The initiator sends a ranging packet and records its transmit timestamp.
2. The responder receives it, waits a known reply time, and sends a response.
3. The initiator receives the response and computes a round-trip time.
4. A second exchange in the opposite direction helps reduce clock-offset and clock-drift error.

One common DS-TWR time-of-flight estimate uses the measured round-trip and reply intervals:

`ToF ~= (Tround1 * Tround2 - Treply1 * Treply2) / (Tround1 + Tround2 + Treply1 + Treply2)`

Then:

`distance = speed of light * ToF`

Further reading: [Symmetrical double-sided two-way ranging](https://en.wikipedia.org/wiki/Symmetrical_double-sided_two-way_ranging).

Key terms:

- **LoS:** direct path between boards is visible.
- **NLoS:** direct path is blocked, so the strongest received path may be reflected or delayed.
- **Bias:** average signed error, `measured - true`.
- **Stability:** how much the range estimate varies over time at a fixed distance.

### 0:15-0:45 - Download SDK and Flash Prebuilt Firmware

1. Go to the [DWM3001CDK product page](https://www.qorvo.com/products/p/DWM3001CDK).
2. Open the **Software** section and download **DW3xxx & QM3xxx SDK**.
3. Unzip the SDK into a folder with a short path and no spaces.
4. Read the SDK quick-start/readme for the DWM3001CDK board.
5. Connect two DWM3001CDK boards by USB.
6. Identify each board's J-Link/debug interface and serial port.
7. Flash one board with the prebuilt DS-TWR initiator firmware.
8. Flash the second board with the prebuilt DS-TWR responder firmware.
9. Open serial terminals for both boards and confirm that DS-TWR messages or range outputs are printing.

Record:

| Board | Role | USB/J-Link ID | Serial Port | Firmware Image |
| --- | --- | --- | --- | --- |
| Board A | Initiator |  |  |  |
| Board B | Responder |  |  |  |

Checkpoint: show the TA one live ranging output.

### 0:45-1:30 - LoS Ranging Measurements

Keep the boards at the same height, with antennas facing the same way when possible. Avoid touching the boards during a recording.

Measure these distances in LoS:

| Trial | True Distance | Duration | Notes |
| --- | --- | --- | --- |
| LoS-1 | 0.5 m | 60 s | Near range |
| LoS-2 | 1.0 m | 60 s | Baseline |
| LoS-3 | 2.0 m | 60 s | Mid range |
| LoS-4 | 3.0 m | 60 s | Longer range |

For each trial:

1. Measure the true distance with tape.
2. Start logging range outputs to a CSV/text file.
3. Collect at least 100 range samples if the demo update rate allows it.
4. Save file names using this pattern:

`group##_los_distance##_trial#.csv`

Suggested CSV columns:

`timestamp_s, true_distance_m, measured_range_m, condition, notes`

### 1:30-2:10 - NLoS Ranging Measurements

Repeat at 1 m and 2 m with controlled blockers:

| Trial | True Distance | Blocker | Duration |
| --- | --- | --- | --- |
| NLoS-1 | 1.0 m | human body between boards | 60 s |
| NLoS-2 | 1.0 m | backpack/cardboard between boards | 60 s |
| NLoS-3 | 2.0 m | human body between boards | 60 s |
| NLoS-4 | 2.0 m | wall/door/furniture if available | 60 s |

Keep the board positions fixed when switching from LoS to NLoS. Only add the blocker. This makes the LoS/NLoS comparison easier to interpret.

Checkpoint question: Does NLoS mostly add noise, a positive bias, dropouts, or all three?

### 2:10-2:50 - Ranging Analysis

For each trial, compute:

- sample count;
- mean measured range;
- bias: `mean(measured - true)`;
- standard deviation;
- MAE: `mean(abs(measured - true))`;
- RMSE: `sqrt(mean((measured - true)^2))`;
- P90 absolute error;
- outlier rate, using `abs(error) > 0.50 m` unless the TA gives another threshold.

Required plots:

1. Range time series for one LoS trial and one NLoS trial at the same distance.
2. Boxplot of absolute error by condition.
3. Error CDF or histogram for LoS vs NLoS.
4. Measured range vs true distance for all LoS trials.

Discussion prompts:

- Which condition was most accurate?
- Which condition was most stable?
- Did NLoS cause a consistent positive or negative error?
- What physical effect could explain that error?
- Would averaging more samples improve accuracy, stability, or both?

### Deeper Thinking: Bandwidth and Ranging Precision

UWB uses very wide bandwidth, and wide bandwidth corresponds to short time-domain pulses. A rough timing intuition is:

`time resolution ~= 1 / bandwidth`

For a bandwidth near `500 MHz`:

`1 / 500 MHz ~= 2 ns`

Since radio waves travel at about `3 x 10^8 m/s`, a `2 ns` timing uncertainty corresponds to about:

`3 x 10^8 m/s * 2 ns ~= 0.6 m`

For radar-style round-trip range resolution, you often see the related expression:

`range resolution ~= c / (2B)`

which gives about `0.3 m` for `B = 500 MHz`.

Discuss with your group:

- Why does increasing bandwidth improve timing resolution?
- In this lab, are we measuring a one-way propagation time, a round-trip time, or an exchange of multiple timestamped packets?
- Is the `0.3-0.6 m` estimate a hard accuracy limit, or only a rough bin-width intuition?
- Practical UWB systems can often report better-than-bin-width range estimates. What signal-processing steps could make that possible?
- Why can NLoS still break the estimate even when the hardware timing resolution is good?

Hint: the receiver is not only choosing a coarse sample bin. It can correlate against a known preamble, estimate the leading edge of the CIR, average repeated measurements, and apply antenna-delay calibration. These steps can improve precision in LoS, while NLoS can still cause a biased estimate because the first detectable path may not be the direct path.

Further reading: [UWB ranging overview](https://en.wikipedia.org/wiki/UWB_ranging).

## Part B: CIR Visualization and UWB Sensing

This part uses TA-provided firmware that streams channel impulse response (CIR) data. If the firmware is not ready, complete Part A thoroughly and treat this section as a design discussion.

### 2:50-3:10 - Flash CIR Streaming Firmware

1. Save your DS-TWR logs before reflashing.
2. Flash the TA-provided CIR firmware to the board pair.
3. Open the serial or USB data stream specified by the TA.
4. Confirm that frames are arriving with CIR samples.

Expected CIR data format may vary. A useful format is:

`timestamp_s, frame_id, sample_index, cir_real, cir_imag, cir_mag, range_m`

If only magnitude is available, use:

`timestamp_s, frame_id, sample_index, cir_mag, range_m`

### 3:10-3:30 - Static CIR Profiles

Collect four static scenes for 30 seconds each:

| Scene | Description |
| --- | --- |
| baseline | no person between boards |
| hand-near-link | hand close to the direct path |
| body-blocking | body blocks or partially blocks the link |
| side-reflector | person or object near the side of the link |

Plot:

- one CIR magnitude profile: x-axis = sample/delay bin, y-axis = magnitude;
- average CIR profile for each static scene;
- difference from baseline: `scene_average - baseline_average`.

Discussion prompts:

- Which delay bins changed the most?
- Does the strongest peak always correspond to the first arriving path?
- Which scene looks most different from baseline?

### 3:30-3:50 - Waterfall Plot

Create a CIR waterfall plot:

- x-axis: delay bin / CIR sample index;
- y-axis: time or frame number;
- color: CIR magnitude or background-subtracted magnitude.

Recommended processing:

1. Build a matrix `X` with shape `frames x delay_bins`.
2. Smooth each delay bin over a short moving average window if the data is noisy.
3. Compute baseline as the average of the first 3-5 seconds with no motion.
4. Plot `abs(X - baseline)`.

Checkpoint question: What does movement look like in the waterfall plot?

### 3:50-4:00 - Mini Sensing Dataset

Collect a small dataset for one application: **hand movement classification**.

Use these labels:

- `none`: no movement;
- `left_right`: hand waves left-right between boards;
- `toward_away`: hand moves toward-away from one board;
- `block_unblock`: hand or body blocks and unblocks the direct path.

Collect 5-10 trials per label. Each trial should last 5 seconds.

Suggested features:

- maximum background-subtracted CIR magnitude;
- delay bin of maximum change;
- total changed energy across all bins;
- early-bin energy;
- late-bin energy;
- energy centroid;
- range mean and range standard deviation during the trial.

Optional classifier:

Train a simple kNN, logistic regression, or decision tree classifier. Report accuracy using a train/test split by trial, not by individual frame, so frames from the same trial do not appear in both train and test.

## Deliverables

Submit one short report per group with:

1. A screenshot or copied log showing DS-TWR ranging output.
2. A table of LoS and NLoS metrics.
3. The four required ranging plots.
4. One CIR profile plot.
5. One CIR waterfall plot.
6. If completed, one table of hand-movement classification features or classifier accuracy.
7. Answers to the discussion prompts.

## What to Conclude

A strong conclusion should mention:

- UWB can give stable range estimates in LoS, but accuracy depends on geometry, calibration, antenna orientation, and logging quality.
- NLoS often increases error because the direct path is blocked or weakened and reflected paths arrive later.
- CIR contains richer information than a single range value because it shows how received energy is distributed over delay.
- Background subtraction turns CIR from a static channel snapshot into a simple motion-sensing signal.

## Extension Ideas

If your group finishes early:

- repeat the experiment on channel 5 and channel 9;
- test whether antenna orientation changes ranging bias;
- compare mean vs median filtering for stable range output;
- train a classifier that is tested on a different group or different location;
- build a simple live waterfall viewer.
