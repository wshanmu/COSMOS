---
layout: page
title: "Lab 6: RFID Reader and Touch Sensor"
description: IoT lab on operating a UHF RFID reader, logging RFID reads over TCP, analyzing RSSI/read-count data, and building a simple three-point RFID touch sensor.
parent: Labs
nav_order: 6
---

# Lab 6: RFID Reader and Touch Sensor **IoT**{: .label .label-iot }

Date: July 13  
Time: 1:00-5:00 PM  
TA: Shanmu Wang  
Hardware: SparkFun Simultaneous RFID Reader - M7E Hecto and passive UHF RFID tags  
Code: [RFID_Lab GitHub repository](https://github.com/wshanmu/RFID_Lab)

## Goals

By the end of this lab, each group should be able to:

- explain the roles of an RFID reader, antenna, passive tag, EPC, RSSI, and read count;
- use Universal Reader Assistant to read passive UHF RFID tags;
- export RFID readings through a TCP port and record them with Python;
- collect repeatable RFID logs under different distance, orientation, and blockage conditions;
- visualize RSSI distributions and read-count changes from RFID logs;
- build and evaluate a simple three-point RFID touch sensor;
- explain why distance, orientation, metal, water-rich objects, and body blockage can make RFID unreliable.

## Team Setup

Form a group of 3 and enter your group information in the [shared Google Sheet](https://docs.google.com/spreadsheets/d/1bWqot4IKFdpzt2ixqVNAvfyYD2ICGSnr11wT_KU4Too/edit?usp=sharing).

Each group should also create a shared Google Slides deck and paste the slide link into the same sheet. Use the slides to record screenshots, plots, photos of your setup, and short answers.

Important requirements:

- At least one group member needs a Windows laptop. Universal Reader Assistant will run on that laptop.
- Other group members may run the Python logging and plotting scripts from their own laptops after connecting to the Windows laptop hotspot.
- Each group will receive one SparkFun Simultaneous RFID Reader and three RFID tags.

Recommended roles:

| Role | Responsibility |
| --- | --- |
| Reader operator | Runs Universal Reader Assistant on the Windows laptop. |
| Python operator | Runs the TCP logging and plotting scripts. |
| Experiment lead | Places tags and keeps the reader/tag geometry consistent. |
| Scribe | Records EPC values, filenames, observations, plots, and slide notes. |

## 1:00-1:20 PM - Repository, Groups, and Hardware

Open VS Code, then open a terminal:

```text
Ctrl + `
```

Clone the lab repository:

```bash
git clone https://github.com/wshanmu/RFID_Lab
cd RFID_Lab
```

Activate the Conda environment from Lab 1 and install the plotting dependency:

```bash
conda activate cosmos-ds
python -m pip install -r requirements.txt
```

Inspect the files:

| File | Purpose |
| --- | --- |
| `URAx64.exe` | Windows installer for Universal Reader Assistant. |
| `reading_from_TCP.py` | Connects to the TCP export stream and saves RFID readings to a log file. |
| `plot_single_log.py` | Plots RSSI distribution and read count for selected EPCs in one log. |
| `plot_multiple_logs.py` | Compares several log files in one folder. |
| `touch_detector_gui.py` | Live three-tag touch detector using baseline RSSI and read rate. |
| `rfid_log_utils.py` | Parses RFID log lines and summarizes data by EPC. |
| `rfid_plot_style.py` | Shared plotting style for the lab figures. |

## 1:20-1:50 PM - Install Universal Reader Assistant and Read Tags

On the Windows laptop, install Universal Reader Assistant:

1. Open the local `RFID_Lab` folder.
2. Double-click `URAx64.exe`.
3. Choose the **Typical** installation option.
4. Launch Universal Reader Assistant after installation.

Connect the SparkFun RFID reader:

1. Connect the reader to the Windows laptop with a USB data cable.
2. Open Universal Reader Assistant.
3. Select the connected reader/serial device.
4. Use the TA-provided settings screenshot for the reader options.
5. Place one RFID tag near the reader.
6. Start reading and confirm that the tag EPC appears.

> Figure placeholder: Universal Reader Assistant connection settings.

Record the EPC values for your three tags:

| Physical tag | EPC | Notes |
| --- | --- | --- |
| Tag 1 |  |  |
| Tag 2 |  |  |
| Tag 3 |  |  |

Checkpoint: show the TA that Universal Reader Assistant can read at least one tag.

## 1:50-2:20 PM - Enable TCP Export and Run Python Logger

Universal Reader Assistant can display tag reads, but for analysis we need to save the stream to files. In this lab, Universal Reader Assistant will export readings through a TCP socket, and the Python scripts will connect to that socket.

A TCP connection lets one program send data to another program over an IP address and port. Here:

- Universal Reader Assistant acts like the TCP server.
- `reading_from_TCP.py` acts like the TCP client.
- The default port is `9055`.
- If Python runs on the same Windows laptop, use host `127.0.0.1`.
- If Python runs on another group member's laptop, use the Windows laptop's hotspot IP address. This is often `192.168.137.1`.

In Universal Reader Assistant:

1. Open the data export or TCP export settings.
2. Enable TCP/socket export.
3. Set the port to `9055`.
4. Keep the export format as plain text.
5. Start reading tags.

> Figure placeholder: Universal Reader Assistant TCP export settings.

### Share the TCP Stream with Group Members

If a teammate will run Python from another laptop:

1. On the Windows laptop, turn on Mobile Hotspot.
2. Ask group members to connect to that hotspot.
3. On the Windows laptop, run `ipconfig` and find the hotspot IPv4 address.
4. On another laptop, you can also run `arp -a` after joining the hotspot to help identify the Windows host.
5. Use that IP address as the `--host` argument.

### Record a Test Log

If Python runs on the same Windows laptop:

```bash
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/baseline --filename test.log
```

If Python runs on a teammate's laptop connected through the Windows hotspot:

```bash
python ./reading_from_TCP.py --host 192.168.137.1 --output-dir logs/baseline --filename test.log
```

The script reads for 15 seconds by default. To change the duration:

```bash
python ./reading_from_TCP.py --host 127.0.0.1 --duration 20 --output-dir logs/baseline --filename test_20s.log
```

The terminal should print lines containing the tag EPC, timestamp, RSSI, and read count. The same text is saved to the log file.

## 2:20-2:55 PM - Distance Experiment

### Question

How do RSSI and read count change as the tag moves farther from the reader?

### Procedure

1. Keep the reader fixed.
2. Use the same three tags for every trial.
3. Put the tags very close to the reader and record 15 seconds.
4. Move the tags to about 25 cm and record 15 seconds.
5. Move the tags to about 50 cm and record 15 seconds.
6. Move the tags to about 100 cm and record 15 seconds.
7. Keep the tag orientation as consistent as possible across trials.

Example commands:

```bash
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/distance --filename next_to.log
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/distance --filename 25cm.log
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/distance --filename 50cm.log
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/distance --filename 100cm.log
```

Replace the example EPCs below with your own three EPC values:

```bash
python ./plot_single_log.py \
  --epcs EPC_TAG_1 EPC_TAG_2 EPC_TAG_3 \
  --log-file logs/distance/25cm.log \
  --title "Distance: 25 cm"
```

```bash
python ./plot_multiple_logs.py \
  --epcs EPC_TAG_1 EPC_TAG_2 EPC_TAG_3 \
  --folder ./logs/distance \
  --title "Distance comparison"
```

If the plot window causes problems, add `--noshow`. The script still saves a PNG file.

Record:

| Distance | Log filename | Which tags were read? | RSSI trend | Read-count trend | Notes |
| --- | --- | --- | --- | --- | --- |
| Very close | `next_to.log` |  |  |  |  |
| 25 cm | `25cm.log` |  |  |  |  |
| 50 cm | `50cm.log` |  |  |  |  |
| 100 cm | `100cm.log` |  |  |  |  |

Questions:

1. Which distance produced the strongest RSSI?
2. Which distance produced the highest read count?
3. Did all tags behave the same way?
4. Did performance degrade gradually or suddenly?

## 2:55-3:25 PM - Orientation Experiment

### Question

How does tag orientation change RFID read reliability?

### Procedure

1. Fix the reader location and choose a distance that worked reliably in the distance experiment.
2. Place all three tags upright/vertical and record 15 seconds.
3. Rotate the tags by 90 degrees so they are horizontal and record 15 seconds.
4. Optionally test a 45-degree orientation if time allows.
5. Use the same host, output folder, and EPC list workflow from the distance experiment.

Example commands:

```bash
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/orientation --filename vertical.log
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/orientation --filename horizontal.log
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/orientation --filename tilted_45deg.log
```

Plot the folder:

```bash
python ./plot_multiple_logs.py \
  --epcs EPC_TAG_1 EPC_TAG_2 EPC_TAG_3 \
  --folder ./logs/orientation \
  --title "Orientation comparison"
```

Record:

| Orientation | Log filename | RSSI trend | Read-count trend | Notes |
| --- | --- | --- | --- | --- |
| Vertical | `vertical.log` |  |  |  |
| Horizontal | `horizontal.log` |  |  |  |
| 45 degrees, optional | `tilted_45deg.log` |  |  |  |

Questions:

1. What did you find when rotating the tags by 90 degrees?
2. Which orientation was most reliable?
3. Which tag changed the most?
4. Why might antenna/tag alignment matter?

## 3:25-3:55 PM - Blockage Experiment

### Question

What happens when the line between the reader and tags is blocked by common objects?

### Procedure

1. Fix the reader location and tag distance.
2. Record a no-blockage baseline.
3. Block the line of sight with a hand and record 15 seconds.
4. Block with a human body and record 15 seconds.
5. Block with a laptop or another metal-rich object and record 15 seconds.
6. Keep the tag locations fixed while only changing the blockage condition.

Example commands:

```bash
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/blockage --filename no_blockage.log
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/blockage --filename hand.log
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/blockage --filename body.log
python ./reading_from_TCP.py --host 127.0.0.1 --output-dir logs/blockage --filename laptop.log
```

Plot the folder:

```bash
python ./plot_multiple_logs.py \
  --epcs EPC_TAG_1 EPC_TAG_2 EPC_TAG_3 \
  --folder ./logs/blockage \
  --title "Blockage comparison"
```

Record:

| Condition | Prediction | Log filename | Observed effect | Notes |
| --- | --- | --- | --- | --- |
| No blockage |  | `no_blockage.log` |  |  |
| Hand |  | `hand.log` |  |  |
| Body |  | `body.log` |  |  |
| Laptop/metal |  | `laptop.log` |  |  |

Questions:

1. Which blockage condition caused the largest RSSI drop?
2. Which condition caused the largest read-count drop?
3. Why are human bodies and metal objects challenging for UHF RFID?
4. Was a low RSSI always the same as a low read count?

## 3:55-4:45 PM - Three-Point RFID Touch Sensor

### Concept

A passive RFID tag can act like a crude touch point because a finger, hand, or object near the tag can change the tag's read strength or read rate. The provided GUI compares live tag readings against a short calibration baseline.

The current detector uses three cues:

- RSSI drop compared with calibration;
- read-rate drop compared with calibration;
- missing reads for a short time window.

### Prepare the Touch Surface

1. Tape three RFID tags to paper, cardboard, or another flat surface.
2. Label them Point 1, Point 2, and Point 3.
3. Keep the points separated enough that a finger can cover one point without covering the others.
4. Record the EPC-to-point mapping.

| Touch point | EPC | Physical location | Notes |
| --- | --- | --- | --- |
| Point 1 |  |  |  |
| Point 2 |  |  |  |
| Point 3 |  |  |  |

### Configure and Run the GUI

Open `touch_detector_gui.py` and edit the configuration block near the top:

```python
HOST = "192.168.137.1"
PORT = 9055

SELECTED_EPCS = [
    "EPC_TAG_1",
    "EPC_TAG_2",
    "EPC_TAG_3",
]
```

Use `HOST = "127.0.0.1"` if the GUI runs on the same Windows laptop as Universal Reader Assistant. Use the Windows hotspot IP if the GUI runs on another laptop.

Run:

```bash
python ./touch_detector_gui.py
```

During the first 10 seconds, do not touch the tags. The GUI is calibrating each tag's baseline RSSI and read rate. After calibration, touch or cover one point at a time and watch whether the GUI marks the correct tag as `TOUCHED`.

### Touch Test Table

| Trial | Point touched | GUI result | Correct? | Notes |
| --- | --- | --- | --- | --- |
| 1 | Point 1 |  |  |  |
| 2 | Point 1 |  |  |  |
| 3 | Point 2 |  |  |  |
| 4 | Point 2 |  |  |  |
| 5 | Point 3 |  |  |  |
| 6 | Point 3 |  |  |  |

### Improve the Detector

If the detector is too sensitive or not sensitive enough, try tuning these constants in `touch_detector_gui.py`:

```python
CALIBRATION_SECONDS = 10.0
WINDOW_SECONDS = 2.0
NO_READ_SECONDS = 1.0

RSSI_DROP_TOUCH_DB = 8.0
RSSI_DROP_RELEASE_DB = 3.0
RATE_DROP_TOUCH_FRACTION = 0.50
RATE_DROP_RELEASE_FRACTION = 0.30
```

Ideas for improving robustness:

1. Tune the RSSI and read-rate thresholds for your own setup.
2. Use per-tag thresholds instead of one global threshold.
3. Use per-tag noise-aware thresholds. For example, during calibration compute the median absolute deviation (MAD) of each tag's RSSI and use:

   ```text
   threshold[tag] = max(8 dB, 3 * MAD[tag])
   ```

   Some tags naturally fluctuate more than others.

4. Use a confidence score instead of hard OR logic:

   ```text
   score = 0.55 * rssi_score + 0.35 * rate_score + 0.10 * missing_score
   ```

   Then display states such as `CLEAR`, `LIKELY TOUCHED`, `TOUCHED`, or `AMBIGUOUS`.

5. Modify `LiveTagMetrics.finish_calibration()` if you want to compute a better baseline.
6. Modify `LiveTagMetrics._update_touch_status()` if you want to change the decision logic.

Questions:

1. Which point was easiest to detect?
2. Which point was hardest to detect?
3. Did touching one point affect the readings from nearby tags?
4. Which change improved the detector most?

## 4:45-5:00 PM - Checkoff and Submission

Show the TA:

1. Universal Reader Assistant reading at least one tag.
2. The EPC values for your three tags.
3. One saved TCP log file.
4. One distance comparison plot.
5. One orientation or blockage comparison plot.
6. The touch detector GUI running with your three selected tags.
7. Your shared slide deck link in the Google Sheet.

Your shared slide deck should include:

1. Group members and hardware setup photo.
2. EPC-to-tag mapping.
3. Distance experiment plot and one-sentence observation.
4. Orientation experiment plot and one-sentence observation.
5. Blockage experiment plot and one-sentence observation.
6. Touch sensor layout photo or sketch.
7. Touch sensor result table.
8. One idea for making the RFID touch sensor more reliable.

## Troubleshooting

| Problem | Likely cause | Fix |
| --- | --- | --- |
| Universal Reader Assistant cannot find the reader | USB cable, driver, or wrong port | Try another data cable, reconnect the reader, restart Universal Reader Assistant, or ask a TA. |
| Tags show in Universal Reader Assistant but Python cannot connect | TCP export is not enabled or host/port is wrong | Confirm TCP export is enabled and use port `9055`. Try `127.0.0.1` on the Windows laptop. |
| Teammate laptop cannot connect | Hotspot/network issue or firewall | Join the Windows hotspot, confirm the host IP with `ipconfig` or `arp -a`, and ask a TA if Windows Firewall blocks the port. |
| Log file is empty | Reader was not reading or TCP stream had no data | Start reading in Universal Reader Assistant before running the Python logger. |
| Plot script says no matching EPC readings were found | EPC list does not match the log | Copy EPCs exactly from the terminal output or Universal Reader Assistant. |
| GUI says `NO BASELINE` | Not enough reads during calibration | Move tags closer, improve tag orientation, or increase `CALIBRATION_SECONDS`. |
| GUI always says `TOUCHED` | Thresholds are too sensitive or baseline was bad | Recalibrate without touching the tags, increase thresholds, or improve tag placement. |

## Core Takeaway

RFID can identify passive tags without line of sight, but reliable RFID systems depend strongly on distance, orientation, materials, antenna geometry, and software filtering. The same reader stream can support both object-identification experiments and simple sensing applications when the data is logged and analyzed carefully.
