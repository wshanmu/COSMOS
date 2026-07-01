---
layout: page
title: "Lab 11: mmWave FMCW Sensing"
description: IoT lab on FMCW mmWave sensing using IWRL6432FSPEVM.
parent: Labs
nav_order: 11
---

# Lab 11: mmWave FMCW Sensing **IoT**{: .label .label-iot }

Date: July 21  
TA: Shanmu  
Format: Groups of 4  
Hardware: One IWRL6432FSPEVM Evaluation Module per group  
Total time: About 4 hours

## Goals

By the end of this lab, each group should be able to:

- explain what an FMCW chirp is and why beat frequency encodes range;
- connect range estimation to the range FFT;
- explain Doppler/velocity estimation from repeated chirps;
- describe how detections become a point cloud;
- explain why multiple TX/RX antennas enable digital beamforming and angle estimation;
- collect first point-cloud data from the IWRL6432FSPEVM using TI's official demo flow.

## Hardware

The IWRL6432FSPEVM is a compact 60 GHz mmWave radar evaluation kit based on TI's IWRL6432 radar device. The official EVM guide describes it as supporting point-cloud output over UART with a 3.3 V input. For this lab, the most important specs are:

| Item | Value |
| --- | --- |
| Radar band | 57-64 GHz |
| Antennas | 2 transmit antennas, 3 receive antennas |
| Field of view | 120 degrees azimuth, 80 degrees elevation |
| Host/data interface used in lab | UART |
| Typical first demo | Motion and presence detection / out-of-box visualization |
| Core processing | Arm M4F plus TI Radar Hardware Accelerator for FFT, log magnitude, and CFAR |

Official references:

- [IWRL6432FSPEVM Evaluation Module User's Guide](https://www.ti.com/lit/pdf/SWRU628)
- [IWRL6432 datasheet](https://www.ti.com/lit/ds/symlink/iwrl6432.pdf)
- [IWRL6432FSPEVM product page](https://www.ti.com/tool/IWRL6432FSPEVM)
- [MMWAVE-L-SDK](https://www.ti.com/tool/MMWAVE-L-SDK)
- [RADAR-TOOLBOX](https://www.ti.com/tool/RADAR-TOOLBOX)

## Recommended Group Roles

Each group of 4 should rotate these roles:

- Hardware lead: wiring, SOP mode, power-cycle checks.
- Software driver: visualizer, COM port, firmware/config flow.
- Experiment lead: target placement and movement.
- Scribe: screenshots, notes, and final answers.

## Before Lab

TA will prepare:

- A radar board per group;
- USB-to-UART converter, micro-USB cable, breakout board, and female-to-female jumper wires.

Each group should prepare:

- one Windows lab PC or VM per group, because TI's official visualizer flow uses Windows `.exe` tools;
- the latest MMWAVE-L-SDK and RADAR-TOOLBOX;
- PuTTY or another serial terminal;
- one simple radar target per group, such as a flat metal reflector or corner reflector if available.

### Serial Terminal Quick Reference

The IWRL6432FSPEVM demo firmware accepts commands and prints status text over a UART serial connection. In this lab, you use the serial terminal to confirm that the board is alive after flashing and to run commands such as `version`.

Conceptually, the data path is:

```text
IWRL6432FSPEVM UART TX/RX -> USB-to-UART adapter -> computer COM port -> PuTTY/screen
```

Use these settings unless the TA gives different values:

| Setting | Value |
| --- | --- |
| Mode | Serial |
| Baud rate | `115200` |
| Data bits | `8` |
| Stop bits | `1` |
| Parity | None |
| Flow control | None |

<details markdown="1">
<summary>Windows: PuTTY</summary>

1. Download PuTTY from the [official PuTTY download page](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).
2. Connect the USB-to-UART adapter.
3. Open **Device Manager** -> **Ports (COM & LPT)** and find the new port, such as `COM6`.
4. Open PuTTY.
5. Select **Serial**.
6. Set **Serial line** to the COM port and **Speed** to `115200`.
7. Click **Open**.
8. Press Enter once. Type `version` and press Enter.

PuTTY's official documentation includes a section on [using a local serial line](https://the.earth.li/~sgtatham/putty/latest/htmldoc/Chapter3.html#using-serial).
</details>

<details markdown="1">
<summary>macOS: Terminal and screen</summary>

The TI visualizer flow is Windows-focused, but macOS can still be useful for reading a serial log.

1. Connect the USB-to-UART adapter.
2. Open **Terminal**.
3. List likely serial ports:

   ```bash
   ls /dev/tty.usb* /dev/tty.usbmodem* 2>/dev/null
   ```

4. Open the port:

   ```bash
   screen /dev/tty.usbserialXXXX 115200
   ```

5. Press Enter once. Type `version` and press Enter.
6. To quit `screen`, press `Ctrl-A`, then `Ctrl-\`, then type `y`.

Apple's [Terminal User Guide](https://support.apple.com/guide/terminal/welcome/mac) is a useful command-line reference.
</details>


## Four-Hour Plan

### 0:00-0:10 - Safety, Roles, and Data Path

Start with the target data path:

`radar chirps -> received echoes -> range/Doppler/angle processing -> UART point-cloud packets -> visualizer plots`

Check that each group has the EVM, breakout board, USB-to-UART adapter, wires, and a PC with the TI tools installed.

### 0:10-0:35 - Wire the EVM

Use the official EVM guide's Section 2.3 setup:

1. Attach the IWRL6432FSP EVM to the included breakout board.
2. Set the USB-to-UART adapter to provide `SYS3V3`.
3. Connect EVM `3V3` to USB-to-UART `SYS3V3`.
4. Connect EVM `GND` to USB-to-UART `GND`.
5. Connect EVM `UART RX` to USB-to-UART `TXD`.
6. Connect EVM `UART TX` to USB-to-UART `RXD`.
7. Use SOP pins to select the mode. For flashing mode, set `SOP0=0` and `SOP1=0`. For functional mode, set `SOP0=1` and `SOP1=0`.
8. Power cycle the board whenever switching SOP modes.

Checkpoint: the group can identify the power, ground, RX/TX, and SOP connections before plugging into the PC.

### 0:35-1:10 - Flash or Verify the Demo Firmware

Follow the official guide's flashing flow:

1. Put the device in flashing mode: `SOP0=0`, `SOP1=0`.
2. Open `MMWAVE_L_SDK_XX_XX_XX_XX/tools/visualizer/visualizer.exe`.
3. Select **Flash**.
4. Select the correct COM port.
5. Select device `XWRL6432`.
6. Upload the prebuilt app image:
   `examples/mmw_demo/motion_and_presence_detection/prebuilt_binaries/xwrL64xx/motion_and_presence_detection_demo.release.appimage`
7. Click **Flash**.
8. Switch to functional mode: `SOP0=1`, `SOP1=0`, then power cycle.
9. Open PuTTY, choose Serial, set the COM port, and set speed to `115200`.
10. Press Enter, type `version`, and confirm the board returns demo/version text.

Deliverable: one screenshot or note showing successful serial connection and version output.

### 1:10-1:50 - Get First Point-Cloud Data

Use the MMWAVE-L-SDK visualizer:

1. Keep the device in functional mode.
2. Open the visualizer and select **Configuration Dashboard**.
3. Enter COM port, baud rate `115200`, and device `XWRL6432`.
4. Select the **High Performance Motion Detection** preset.
5. Click **Send Config to Device**.
6. Open plots for range profile and point-cloud views such as X-Y and Y-Z.
7. Place a reflector at 0.5 m, 1 m, and 1.5 m. Record what changes.

Deliverable: one range profile screenshot and one point-cloud screenshot.

### 1:50-2:25 - FMCW Range Estimation

Mini-lesson:

- An FMCW radar transmits chirps whose frequency ramps over time.
- A delayed echo mixes with the current transmit signal and produces a beat frequency.
- For a chirp slope `S`, beat frequency `f_b`, and speed of light `c`, a simplified range estimate is `R = c f_b / (2S)`.
- The range FFT separates beat frequencies into range bins.

Activity:

1. Keep the same configuration.
2. Move one strong reflector away from the radar in steps.
3. Observe the range-profile peak shift.
4. Note how clutter or nearby people affect the plot.

Checkoff question: Why does a farther object produce a different beat frequency?

### 2:25-3:00 - Doppler and Velocity

Mini-lesson:

- Repeated chirps allow the radar to compare phase changes over time.
- Motion toward or away from the radar creates Doppler shift.
- A Doppler FFT across chirps separates stationary and moving targets.

Activity:

1. Have one student slowly walk toward the radar, then away from it.
2. Compare the plotted points and motion signatures.
3. Repeat with a hand wave near the sensor.

Checkoff question: Why is a stationary chair easier to remove than a moving hand?

### 3:00-3:35 - Point Clouds and Detection

Mini-lesson:

- The radar processing chain builds a radar cube across range, Doppler, and antenna channels.
- Detection logic such as CFAR keeps strong candidate returns.
- Each detected target can be reported as a point with coordinates and signal strength.

Activity:

1. Create three scenes: empty space, one reflector, and two people moving.
2. Capture screenshots of the point cloud for each scene.
3. Record which scene creates the most false or unstable points.

Checkoff question: What information is lost when raw radar samples are reduced to a point cloud?

### 3:35-3:55 - Digital Beamforming and Angle

Mini-lesson:

- Multiple TX and RX antennas form a virtual antenna array.
- Different arrival angles create different phase patterns across antennas.
- Digital beamforming estimates angle by testing which steering direction best matches the measured phase pattern.

Activity:

1. Move a reflector left, center, and right at roughly the same range.
2. Observe whether the point cloud shifts in X-Y.
3. If elevation plotting is stable, repeat with a higher/lower reflector.

Checkoff question: Why do more antenna channels usually improve angle estimation?

### 3:55-4:00 - Wrap-Up

Each group submits:

- serial connection/version evidence;
- one range-profile screenshot;
- one point-cloud screenshot;
- short answers to the four checkoff questions;
- one limitation they noticed in the point cloud.

## Extension

The IWRL6432 datasheet lists RDIF as the radar data interface for raw ADC sample capture. Raw ADC capture is not the main target of this 4-hour lab, but advanced groups can compare the point-cloud pipeline with what would be needed to process ADC samples manually: range FFT, Doppler FFT, CFAR, and angle estimation.
