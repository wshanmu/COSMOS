---
layout: page
title: "Lab 3: WiFi, Wireshark, and CSI Collection"
description: IoT lab on WiFi fundamentals, ESP32 CSI extraction, and presence sensing.
parent: Labs
nav_order: 3
---

# Lab 3: WiFi, Wireshark, and CSI Collection **IoT**{: .label .label-iot }

Date: July 8  
Time: 1:00-5:00 PM  
TA: Shanmu Wang  
Hardware: three ESP32 boards per group: `softAP`, `injector`, and `sniffer`

## Goals

By the end of this lab, each group should be able to:

- explain WiFi frames, channels, RSSI (received signal strength indicator), OFDM subcarriers, and channel state information (CSI);
- optionally use Wireshark to inspect a captured WiFi packet trace;
- flash or verify three ESP32 firmware roles: access point, frame injector, and CSI sniffer;
- identify and record ESP32 serial ports, MAC addresses, and WiFi channels;
- visualize CSI amplitude over time and across subcarriers;
- implement a simple variance-based CSI motion detector;
- explain why WiFi sensing depends on multipath, geometry, motion, and interference.

## Lab Code

Use the WiFi lab repository for the firmware projects, Python tools, and example packet capture:

[https://github.com/wshanmu/wifi_lab](https://github.com/wshanmu/wifi_lab)

Clone it once:

```bash
git clone https://github.com/wshanmu/wifi_lab.git
cd wifi_lab
```

Before lab, update to the latest version:

```bash
git pull
```

The main folders are:

| Path | Purpose |
| --- | --- |
| `softAP/` | ESP-IDF firmware for the ESP32 access point. |
| `injector/` | ESP-IDF firmware that transmits spoofed WiFi frames as the CSI stimulus. |
| `sniffer/` | ESP-IDF firmware that listens for the injector frames and streams CSI over USB serial. |
| `tools/` | Python live CSI plot and presence-detection tools. |
| `wireshark_example.pcap` | Example WiFi packet capture for the Wireshark demo. |

## System Overview

The lab uses three ESP32 roles.

```text
softAP creates WiFi link context
injector sends spoofed 802.11 frames
sniffer captures CSI from those frames and streams it over USB serial
```

CSI is richer than RSSI. RSSI is one received-power number. CSI gives amplitude and phase information for many OFDM subcarriers, so it changes when people move and alter the multipath reflections in the room.

## Hardware and Board Roles

Each group will use three ESP32-based boards. An ESP32 is a WiFi/Bluetooth microcontroller. A microcontroller, or MCU, is a small computer designed to run firmware directly, control hardware pins, talk to sensors, and react to events in real time.

| Role | Board | ESP-IDF target | Purpose |
| --- | --- | --- | --- |
| `softAP` | [M5Stack Core2 ESP32 IoT Development Kit](https://shop.m5stack.com/products/m5stack-core2-esp32-iot-development-kit) | `esp32` | Creates the WiFi access point and fixes the channel used by the lab. |
| `sniffer` | [ESP32-CAM](https://lastminuteengineers.com/getting-started-with-esp32-cam/) | `esp32` | Captures packets in promiscuous mode and streams CSI over USB serial. |
| `injector` | [ESP32-C3-Mini](https://www.amazon.com/ESP32-C3-Development-Bluetooth-Single-Core-Processor/dp/B0D4QD19BB) | `esp32c3` | Sends repeated 802.11 frames that create a controllable CSI signal source. |

Use the ESP32-CAM as the sniffer unless a TA tells you otherwise. The current sniffer firmware and CSI data path are tested on the classic ESP32 target, not on the ESP32-C3-Mini.

## Group and Shared Log

Form groups of three. A Windows laptop is not required for this lab.

Use the [shared Google Sheet](https://docs.google.com/spreadsheets/d/17h-pmffmxZH8AygtE6UqEtIz8x_9kswIOpQA3G9yL8o/edit?gid=0#gid=0) to record:

- group name and member names;
- assigned board roles and serial ports;
- SoftAP MAC address and WiFi channel;
- injector fake MAC address;
- experiment notes and final short-report link.

After groups are formed, the TA will distribute one kit per group: one `softAP` board, one `injector` board, one `sniffer` board, and the needed USB cables.

## Before You Start

Each group should have one kit with:

- one [M5Stack Core2 ESP32 IoT Development Kit](https://shop.m5stack.com/products/m5stack-core2-esp32-iot-development-kit) as the `softAP`;
- one [ESP32-C3-Mini](https://www.amazon.com/ESP32-C3-Development-Bluetooth-Single-Core-Processor/dp/B0D4QD19BB) as the `injector`;
- one [ESP32-CAM](https://lastminuteengineers.com/getting-started-with-esp32-cam/) as the `sniffer`;
- USB data cables, not charge-only cables;
- at least one laptop with VS Code installed;
- the `cosmos-ds` Conda environment from Lab 1;
- the WiFi lab repository cloned locally.

If the boards are already pre-flashed by the TA, still read the firmware steps so you understand what each board is doing. You may only need to verify the serial output.

## ESP-IDF Setup

ESP-IDF is Espressif's official toolchain for building, flashing, and monitoring ESP32 firmware. Set it up in this lab; the same setup will also be useful for the July 9 IMU lab.

1. Install or update [Visual Studio Code](https://code.visualstudio.com/).
2. In VS Code, install the official **ESP-IDF** extension.
3. Open the command palette:
   - macOS: `Command+Shift+P`
   - Windows/Linux: `Ctrl+Shift+P`
4. Run `ESP-IDF: Open ESP-IDF Installation Manager`.
5. Install ESP-IDF and its tools using the installer.
6. Run `ESP-IDF: Select Current ESP-IDF Version`.
7. Run `ESP-IDF: Doctor Command` and fix any reported setup issues.

### Verify ESP-IDF With Hello World

Before flashing the WiFi lab firmware, verify the full build, flash, and monitor pipeline with ESP-IDF's `hello_world` example.

1. Open the VS Code command palette.
2. Run `ESP-IDF: New Project`.
3. Choose the `hello_world` example project.
4. Save it outside the `wifi_lab` repository, for example in a temporary `esp-idf-test` folder.
5. Select the target:
   - use `esp32` if testing the M5Stack Core2 or ESP32-CAM;
   - use `esp32c3` if testing the ESP32-C3-Mini.
6. Select the serial port for the connected board.
7. When flashing, use UART as the flashing method.
8. Run `ESP-IDF: Build, Flash and Start a Monitor on Your Device`.

![Configure hello world project step 1]({{ site.baseurl }}/assets/images/wifi-lab/configure_hello_world_project_1.jpg)

![Configure hello world project step 2]({{ site.baseurl }}/assets/images/wifi-lab/configure_hello_world_project_2.jpg)

![Configure hello world project step 3]({{ site.baseurl }}/assets/images/wifi-lab/configure_hello_world_project_3.jpg)

The monitor should show the `hello_world` output and then restart countdown messages.

![Hello world monitor output]({{ site.baseurl }}/assets/images/wifi-lab/monitor_device_result_for_hello_world.jpg)

When flashing a board, open that board's firmware folder in VS Code and select the correct target:

| Folder | Board role | Target |
| --- | --- | --- |
| `softAP/` | M5Stack Core2 SoftAP | `esp32` |
| `injector/` | ESP32-C3-Mini injector | `esp32c3` |
| `sniffer/` | ESP32-CAM sniffer | `esp32` |

Then run:

1. `ESP-IDF: Select Port to Use`.
2. `ESP-IDF: Build your Project`.
3. `ESP-IDF: Flash your Project`.
4. `ESP-IDF: Monitor your Device`.

Do not erase flash or overwrite code outside the lab repository unless a TA asks you to.

Official references:

- [ESP-IDF Extension for VS Code installation guide](https://docs.espressif.com/projects/vscode-esp-idf-extension/en/latest/installation.html)
- [ESP-IDF Get Started guide for ESP32](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/get-started/index.html)
- [ESP-IDF Get Started guide for ESP32-C3](https://docs.espressif.com/projects/esp-idf/en/stable/esp32c3/get-started/index.html)

## Python Tool Setup

Open a terminal in the repository:

```bash
cd wifi_lab
```

Activate the Conda environment from Lab 1 and install the WiFi lab packages:

```bash
conda activate cosmos-ds
python -m pip install -r tools/requirements.txt
```

Check the plotting tool without hardware:

```bash
python tools/plot_csi_serial.py --demo-signal
```

A window titled `ESP32 CSI Live Amplitude` should open.

## Find Serial Ports

Use your actual serial port in later commands.

macOS:

```bash
ls /dev/cu.* 2>/dev/null
```

Linux:

```bash
ls /dev/ttyUSB* /dev/ttyACM* 2>/dev/null
```

Windows:

```text
Open Device Manager -> Ports (COM & LPT), then look for the new COM port.
```

Useful examples:

- macOS: `/dev/cu.usbserial-5B1F0080901`
- Linux: `/dev/ttyUSB0`
- Windows: `COM5`

In VS Code, the ESP-IDF status bar also shows the selected serial port and target. Confirm these before flashing.

![ESP-IDF selected port and target]({{ site.baseurl }}/assets/images/wifi-lab/image-2.png)

## Four-Hour Plan

### 1:00-1:10 PM - Goals and Repository Clone

The TA will first explain the lab goals, the final expected result, and the three ESP32 roles.

Then clone the lab repository:

```bash
git clone https://github.com/wshanmu/wifi_lab.git
cd wifi_lab
```

If you already cloned the repository, update it:

```bash
git pull
```

### 1:10-1:25 PM - Groups and Kit Distribution

Form groups of three. Add your group information to the [shared Google Sheet](https://docs.google.com/spreadsheets/d/17h-pmffmxZH8AygtE6UqEtIz8x_9kswIOpQA3G9yL8o/edit?gid=0#gid=0).

Each group will receive one kit:

- one `softAP` board;
- one `injector` board;
- one `sniffer` board;
- USB data cables.

Before moving on, record your group name and member names in the shared sheet.

### 1:25-2:15 PM - Guided ESP-IDF Setup and Hello World Flash

The TA will walk through ESP-IDF setup step by step. Each group should confirm:

1. VS Code can see the ESP-IDF extension commands.
2. ESP-IDF Doctor reports a usable installation.
3. One board can build, flash, and monitor the `hello_world` example.
4. The serial monitor shows readable `hello_world` output.

Use the hello-world verification steps above before flashing the WiFi lab firmware. This confirms that the USB cable, driver, ESP-IDF target, flash method, and serial monitor are working.

### 2:15-2:45 PM - Slides, Wireshark Demo, and Expected Result

The TA will use slides to introduce:

- **WiFi frame:** one structured packet on the wireless link.
- **Channel:** the frequency slice the radio uses, such as 2.4 GHz channel 1.
- **OFDM subcarrier:** a narrow frequency bin inside the WiFi channel.
- **RSSI:** one scalar received signal strength value.
- **CSI:** per-subcarrier channel response. We will use CSI amplitude for sensing.
- **Multipath:** reflected signal paths from walls, furniture, and people.

Key idea: when a person moves near the injector-sniffer link, the wireless channel changes. That motion can show up as higher CSI variance.

The TA will also show a Wireshark demo and the expected end-to-end result:

1. SoftAP is visible as a WiFi network.
2. Injector prints `Starting injection...`.
3. Sniffer prints CSI lines with timestamp, RSSI, MAC address, and CSI values.
4. The Python plot shows a live CSI heatmap.
5. The motion detector score increases when someone moves near the link.

To inspect the provided example capture yourself, install [Wireshark](https://www.wireshark.org/download.html), then open:

```bash
wireshark wireshark_example.pcap
```

If that command does not work, open Wireshark first and use **File > Open**.

Find one beacon or data frame and inspect:

- frame type;
- source MAC address;
- destination MAC address;
- sequence number;
- channel or radio metadata if available.

Optional live capture: on Linux, with a compatible WiFi adapter and permission to monitor the local lab channel, you can run:

```bash
sudo ./wifi_monitor_capture.sh wlan0 1
```

Replace `wlan0` and `1` with your wireless interface and lab channel. Only capture traffic on networks and spectrum you are authorized to monitor.

### 2:45-3:15 PM - Configure or Verify the SoftAP Board

The SoftAP board creates the WiFi network context for the link.

1. Open the `softAP/` folder in VS Code.
2. Connect the board labeled `softAP`.
3. Identify its serial port.
4. Open the ESP-IDF Command Palette with `Ctrl+Shift+P` or `Command+Shift+P`.

![ESP-IDF command palette]({{ site.baseurl }}/assets/images/wifi-lab/image.png)

5. Open `ESP-IDF: SDK Configuration Editor (Menuconfig)`.
6. Search for WiFi settings and set the WiFi channel. Use channel `1` unless the TA assigns another channel.

You can also use the gear icon in the ESP-IDF toolbar to open configuration tools.

![ESP-IDF toolbar configuration shortcut]({{ site.baseurl }}/assets/images/wifi-lab/image-9.png)

![SoftAP channel setting]({{ site.baseurl }}/assets/images/wifi-lab/image-8.png)

7. Run `ESP-IDF: Build, Flash and Start a Monitor on Your Device`.
8. Record the SoftAP MAC address and channel from the monitor output.

![SoftAP expected monitor output]({{ site.baseurl }}/assets/images/wifi-lab/image-4.png)

Record:

| Item | Value |
| --- | --- |
| SoftAP serial port |  |
| SoftAP MAC address |  |
| WiFi channel |  |

After the SoftAP is running, it can be powered from a USB power adapter.

SoftAP verification: use a phone or laptop to scan for the SoftAP SSID. You do not need to connect to the internet through it. Just confirm that the SSID appears and record the channel shown in the monitor output.

### 3:15-3:40 PM - Configure or Verify the Injector Board

The injector board sends spoofed 802.11 frames to create packets that the sniffer can capture.

1. Open the `injector/` folder in VS Code.
2. Connect the board labeled `injector`.
3. Open `main/injector.c`.
4. Set bytes `4-9` and `16-21` in the packet to the SoftAP MAC address.

![Injector packet destination bytes]({{ site.baseurl }}/assets/images/wifi-lab/image-3.png)

5. Set bytes `10-15` to a fake transmitter MAC address. Use a unique value for your group and write it down.

![Injector fake MAC bytes]({{ site.baseurl }}/assets/images/wifi-lab/image-5.png)

6. Set the injector WiFi channel to the same channel as the SoftAP:

```c
ESP_ERROR_CHECK(esp_wifi_set_channel(YOUR_CHANNEL, 0));
```

7. Run `ESP-IDF: Build, Flash and Start a Monitor on Your Device`.

Expected injector output includes `Starting injection...`.

![Injector expected monitor output]({{ site.baseurl }}/assets/images/wifi-lab/image-6.png)

Record:

| Item | Value |
| --- | --- |
| Injector serial port |  |
| Injector fake MAC address |  |
| Injector channel |  |

### 3:40-4:05 PM - Configure or Verify the Sniffer Board

The sniffer board listens on the same channel and streams CSI reports over USB serial.

1. Open the `sniffer/` folder in VS Code.
2. Connect the board labeled `sniffer`.
3. Open `main/sniffer.c`.
4. Set the injector fake MAC address:

```c
#define INJECTOR_SPOOFED_MAC "AA:BB:BB:BB:BB:BB"
```

5. Set the sniffer channel to the same channel:

```c
ESP_ERROR_CHECK(esp_wifi_set_channel(YOUR_CHANNEL, 0));
```

6. Run `ESP-IDF: Build, Flash and Start a Monitor on Your Device`.
7. Confirm that the monitor prints CSI lines with timestamp, RSSI, address, and many integer CSI values.

![Sniffer expected CSI output]({{ site.baseurl }}/assets/images/wifi-lab/image-7.png)

Record:

| Item | Value |
| --- | --- |
| Sniffer serial port |  |
| Sniffer listening MAC filter |  |
| Sniffer channel |  |

Checkpoint: show the TA one live CSI line before moving on.

### 4:05-4:25 PM - Live CSI Visualization

Run the live CSI plot from the repository root:

```bash
python tools/plot_csi_serial.py --port SNIFFER_PORT --baud 115200
```

Replace `SNIFFER_PORT` with your sniffer serial port.

Observe:

- heatmap: subcarrier index vs. time;
- selected subcarrier amplitude over time;
- RSSI in the status line.

Try these scenes:

| Scene | What to do |
| --- | --- |
| static | Keep the injector-sniffer area empty and still. |
| walk | Walk through the link path. |
| wave | Wave one arm near the link path. |

Question: which subcarriers visibly change when someone moves?

Some subcarriers may look zero, invalid, or almost static. This is expected. The CSI buffer includes values from different long training fields, and the useful subcarrier range depends on packet type, bandwidth, and driver configuration. Some indices can also be guard/DC/pilot-related positions or invalid hardware words. Focus your detector on the subcarriers that respond clearly to motion instead of assuming every index is equally useful.

Useful reading:

- [Espressif Wi-Fi Channel State Information](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/wifi-driver/wifi-vendor-features.html#wi-fi-channel-state-information)
- [Espressif Wi-Fi CSI configuration steps](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/wifi-driver/wifi-vendor-features.html#wi-fi-channel-state-information-configure)

### 4:25-4:50 PM - Presence Detection Coding Task

Open:

```text
tools/csi_presence_detect.py
```

Complete `MotionDetector.update()`.

The intended logic is:

1. Append the newest CSI amplitude vector to `self.history`.
2. Wait until at least two vectors are available.
3. Stack recent vectors into a matrix.
4. Compute variance over time for each subcarrier.
5. Average those variances into one motion score.
6. Return `motion = score > self.threshold`.

Run live:

```bash
python tools/csi_presence_detect.py --port SNIFFER_PORT --threshold 2.0 --window-size 20 --log present.txt
```

You can also test without hardware:

```bash
python tools/csi_presence_detect.py --demo-signal
```

Experiment:

- lower the threshold and watch for false positives;
- raise the threshold and watch for missed motion;
- change `--window-size` and observe latency vs. stability.

### 4:50-5:00 PM - Checkoff and Discussion

Show the TA:

1. SoftAP MAC address and channel.
2. Injector fake MAC address.
3. One live sniffer CSI line.
4. Live CSI heatmap or demo-signal plot.
5. Presence detector running with your completed `MotionDetector.update()`.
6. One threshold/window setting that worked reasonably well.

## Deliverables

Submit one short group note and add the link to the shared Google Sheet. Include:

1. Board table: serial port, role, MAC address, and channel.
2. Screenshot or photo of one live CSI line.
3. Screenshot of the CSI visualization or demo signal.
4. Your completed `MotionDetector.update()` code.
5. Short answers:
   - What changed in CSI when someone moved?
   - What threshold did you choose?
   - What caused false positives or false negatives?
   - How would room layout or other WiFi traffic affect the result?

## Troubleshooting

| Problem | Likely Cause | Fix |
| --- | --- | --- |
| Board not visible as a serial port | Charge-only cable or missing USB driver | Try a known data cable; check Device Manager or `/dev/cu.*`; ask a TA. |
| Flash fails | Wrong target, wrong port, or board not in bootloader mode | Select the right port; hold BOOT during connect if needed; ask a TA before erasing. |
| Sniffer prints no CSI | MAC filter or channel mismatch | Confirm injector fake MAC and channel match the sniffer configuration. |
| Live plot opens but stays blank | Wrong serial port or sniffer not streaming | Recheck `SNIFFER_PORT` and monitor output. |
| PyQt/PyQtGraph error | Python packages missing | Activate `cosmos-ds` and run `python -m pip install -r tools/requirements.txt`. |
| Detector never triggers | Threshold too high or too little motion near link | Lower `--threshold`, move closer to the injector-sniffer path, or increase motion. |
| Detector always triggers | Threshold too low or noisy environment | Raise `--threshold`, reduce nearby movement, or reposition boards. |

## Optional Extension

If time remains, record a 60-second still-person CSI log and look for small periodic amplitude changes from breathing. This is harder than motion detection because breathing creates a much smaller signal, but it uses the same CSI pipeline.
