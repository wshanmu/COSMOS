---
layout: page
title: "Lab 6: RFID Reader and Touch Sensor"
description: IoT lab on operating a UHF RFID reader, testing read reliability, and building a simple three-point RFID touch sensor.
parent: Labs
nav_order: 6
---

# Lab 6: RFID Reader and Touch Sensor **IoT**{: .label .label-iot }

Date: July 13  
Time: 1:00-5:00 PM  
TA: Shanmu Wang  
Hardware: SparkFun Simultaneous RFID Reader - M7E Hecto and passive UHF RFID tags  

## Goals

By the end of this lab, each group should be able to:

- explain the roles of an RFID reader, antenna, tag, and EPC;
- connect and operate a UHF RFID reader;
- record RFID tag reads in a repeatable lab log;
- test how distance, orientation, materials, and multiple tags affect RFID reliability;
- explain why metal, water-rich objects, tag placement, and reader geometry can make RFID unreliable;
- build and evaluate a simple three-point RFID touch sensor.

## Team Kit

Each group should have:

- one SparkFun Simultaneous RFID Reader - M7E Hecto;
- one USB-C data cable;
- several passive UHF RFID tags;
- tape, paper or cardboard, and a ruler or measuring tape.

## Recommended Group Roles

- Hardware lead: connects the reader, manages cable/port setup, and keeps the reader position fixed.
- Reader operator: runs the assigned software, example script, or serial interface.
- Experiment lead: places tags, manages distance/orientation/material trials, and keeps timing consistent.
- Scribe: records EPC values, observations, troubleshooting, and final answers.

Rotate roles after the baseline test if time allows.

## Safety and Setup Notes

- Use the reader only with the TA-provided antenna/board setup.
- Keep liquids away from the electronics, even when testing water-rich objects.
- Do not change hardware jumpers or reader power settings unless a TA asks you to.
- Keep your reader position fixed during a trial. Moving both the reader and tag at the same time makes the data hard to interpret.

## 1:00-1:25 PM - Hardware Setup and First Read

### Driving Question

How can a computer identify an object without seeing it or touching it?

### Setup Checklist

| Step | Complete? | Notes |
| --- | --- | --- |
| Reader board visually inspected |  |  |
| USB-C data cable connected |  |  |
| Computer detects the device |  |  |
| Correct port or device selected |  |  |
| Reader software or example opened |  |  |
| Reader produces output |  |  |
| Known tag detected |  |  |

### Procedure

1. Record your team name and reader number.
2. Inspect the reader board. Identify the USB-C port, onboard antenna area, power indicators, and labeled connectors.
3. Connect the reader to the computer using the USB-C cable.
4. Confirm that the computer recognizes the reader.
5. Open the assigned reader software, example program, or serial interface.
6. Place one known RFID tag near the reader.
7. Record whether the reader detects the tag.

### Troubleshooting Log

| Problem observed | Evidence | Attempted fix | Result |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |

Checkpoint: show the TA one detected RFID tag.

## 1:25-1:45 PM - RFID Concepts

### Key Vocabulary

- **RFID:** Radio Frequency Identification.
- **Reader:** Device that sends and receives radio signals.
- **Antenna:** Part of the system that radiates and receives RF energy.
- **Tag:** Small device attached to an object to be identified.
- **EPC:** Electronic Product Code, a digital identifier stored on many UHF RFID tags.
- **UHF:** Ultra High Frequency.

### Activity

1. List three systems that track physical objects.
2. Compare a barcode or QR code with an RFID tag.
3. Use your reader to detect a tag without direct line of sight.
4. Write down one advantage and one limitation of RFID.

Discussion questions:

- Why might RFID be useful in a warehouse?
- Why might a barcode still be better in some situations?
- What could make an RFID tag hard to read?
- Does wireless always mean reliable?

## 1:45-2:15 PM - Baseline Reader Operation

### Driving Question

What does the RFID reader actually report when it detects a tag?

### Procedure

1. Place one tag near the onboard antenna.
2. Record the EPC value shown by the reader.
3. Repeat for at least three tags.
4. Place two or three tags near the reader and observe the output.
5. Record a baseline result for your reader.

### Data Table

| Physical tag name | EPC value | Read successfully? | Notes |
| --- | --- | --- | --- |
| Tag A |  |  |  |
| Tag B |  |  |  |
| Tag C |  |  |  |

Complete this baseline statement:

> Our reader number was ________. Using ________ software, we detected ________ tags at approximately ________ cm.

Reflection question: how can you tell whether the reader detected a new tag or repeated the same tag?

## 2:15-2:50 PM - Distance and Signal Strength

### Driving Question

How does distance affect whether an RFID tag can be read?

### Procedure

1. Tape a ruler or measuring tape to the desk starting at the antenna face.
2. Choose one RFID tag.
3. Place the tag at the first distance.
4. Run the reader for 10 seconds.
5. Count successful reads or record signal strength if available.
6. Repeat for each distance.
7. Repeat the full trial at least twice.
8. Graph the result if time allows.

### Data Table

| Distance from antenna | Trial 1 reads | Trial 2 reads | Trial 3 reads | Average reads | Notes |
| --- | ---: | ---: | ---: | ---: | --- |
| Touching |  |  |  |  |  |
| About 0.5 feet |  |  |  |  |  |
| About 1 foot |  |  |  |  |  |
| Farther distance chosen by group |  |  |  |  |  |

Analysis questions:

- At what distance did reading become unreliable?
- Did the tag suddenly stop working, or did reliability gradually decrease?
- Why might two teams get different results using similar readers?
- How would increasing reader power affect your results?

## 2:50-3:20 PM - Orientation and Tag Placement

### Driving Question

Why does the same RFID tag read well in one position but poorly in another?

### Procedure

1. Choose one fixed distance that worked reliably in the distance test.
2. Place the tag flat facing the antenna and record reads for 10 seconds.
3. Repeat with the tag tilted at about 45 degrees.
4. Repeat with the tag edge-on at about 90 degrees.
5. Repeat with the tag flat but rotated sideways.
6. Try the same test with a second tag shape if available.

### Data Table

| Tag orientation | Reads in 10 seconds | Reliable? | Notes |
| --- | ---: | --- | --- |
| Flat facing antenna |  |  |  |
| 45 degrees |  |  |  |
| Edge-on, 90 degrees |  |  |  |
| Flat but rotated sideways |  |  |  |

Analysis questions:

- Which orientation worked best?
- Which orientation worked worst?
- Did the results match your prediction?
- How should a store place tags on products to improve reliability?

## 3:20-3:50 PM - Materials and Interference

### Driving Question

What materials make RFID easier or harder to read?

### Procedure

1. Predict whether each material will allow reliable reading.
2. Place the material between the tag and reader.
3. Run the reader for 10 seconds and record results.
4. Try placing the tag directly on or near metal.
5. Compare your results to your predictions.

### Data Table

| Material or condition | Prediction | Reads in 10 seconds | Result | Notes |
| --- | --- | ---: | --- | --- |
| No obstruction |  |  |  |  |
| Paper |  |  |  |  |
| Cardboard |  |  |  |  |
| Plastic |  |  |  |  |
| Water bottle |  |  |  |  |
| Laptop |  |  |  |  |
| Tag directly on metal |  |  |  |  |

Analysis questions:

- Which materials had little effect?
- Which materials caused the biggest problem?
- Why might tagging a metal water bottle be difficult?
- What could engineers change to solve this problem?

Complete this recommendation:

> If I needed to tag an object made of ________, I would place the RFID tag ________ because ________.

## 3:50-4:45 PM - Three-Point RFID Touch Sensor

### Driving Question

Can three RFID tags be used as a simple three-point touch sensor?

### Sensor Concept

Each RFID tag represents one input point. When a user places a finger, hand, card, or object over a marked point, the reader should detect the tag assigned to that point. Your goal is to build and test a three-point sensor that can tell which point is being touched.

Label the points:

- Point 1
- Point 2
- Point 3

### Design Requirements

Your sensor must:

- use exactly three RFID tags;
- give each tag a known physical location;
- record the EPC value for each point;
- keep the three points far enough apart to compare them;
- use a repeatable touch action for every trial;
- decide how to handle repeated reads of the same tag;
- report whether the touched point matched the tag detected by the reader.

### Procedure

1. Choose three RFID tags and record their EPC values.
2. Tape the three tags to a sheet of paper, cardboard, or another flat surface.
3. Label the locations Point 1, Point 2, and Point 3.
4. Place the reader in a fixed position near the touch surface.
5. Test each point without touching it and record any background reads.
6. Touch Point 1 using the same touch method for each trial and run the reader for 5 seconds.
7. Repeat for Point 2 and Point 3.
8. Run at least three trials for each point.
9. Revise the tag spacing, reader position, or touch method if the reader detects the wrong point or misses a touch.
10. Repeat the test after one design change.

### Tag Map

| Touch point | Physical location | EPC value | Notes |
| --- | --- | --- | --- |
| Point 1 |  |  |  |
| Point 2 |  |  |  |
| Point 3 |  |  |  |

### Test Data

| Trial | Point touched | EPC detected first | Other EPCs detected? | Correct point detected? | Notes |
| --- | --- | --- | --- | --- | --- |
| 1 | Point 1 |  |  |  |  |
| 2 | Point 1 |  |  |  |  |
| 3 | Point 1 |  |  |  |  |
| 4 | Point 2 |  |  |  |  |
| 5 | Point 2 |  |  |  |  |
| 6 | Point 2 |  |  |  |  |
| 7 | Point 3 |  |  |  |  |
| 8 | Point 3 |  |  |  |  |
| 9 | Point 3 |  |  |  |  |

Analysis questions:

- Which point was detected most reliably?
- Which point was detected least reliably?
- Did touching one point ever cause another point to be read?
- How did tag spacing affect the sensor?
- How could software decide which point was touched if several EPCs appeared?

## 4:45-5:00 PM - Checkoff and Reflection

Show the TA:

1. a setup record showing one known tag detected;
2. EPC values for at least three tags;
3. distance, orientation, and materials data tables;
4. a sketch or photo of the three-point sensor layout;
5. the EPC-to-point map and test table;
6. one design change you made after testing.

Final reflection:

1. What was the most reliable RFID reading condition you tested?
2. What was the least reliable condition?
3. What surprised you most?
4. What is one real-world system where RFID would be useful?
5. What is one real-world system where RFID might be a poor choice?

## Core Takeaway

RFID can identify objects without line of sight, but reliable RFID systems require careful choices about distance, orientation, materials, antenna placement, software filtering, and testing.
