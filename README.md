# High-Speed Camera System
An embedded system for visualizing fast UV-triggered polymerization reactions in slow motion.

## Table of Contents
- [High-Speed Camera System](#high-speed-camera-system)
  - [Introduction](#introduction)
  - [System Architecture](#system-architecture)
  - [System Components](#system-components)
  - [Optimizing Capture Speed](#optimizing-capture-speed)
  - [Frame Processing and Output](#frame-processing-and-output)
  - [System Testing Update](#system-testing-update)
  - [Outdoor Lighting Test](#outdoor-lighting-test)
    - [Sample Captures](#sample-captures)
  - [Exhibit Testing – First On-Site Capture Attempts](#exhibit-testing--first-on-site-capture-attempts)
  - [Mouse Based Triggering of Video Capture](#mouse-based-triggering-of-video-capture)
  - [Prototype Assembly and Function](#prototype-assembly-and-function)
  - [Arduino to Raspberry Pi Trigger Interface](#arduino-to-raspberry-pi-trigger-interface)
  - [Next Steps](#next-steps)
 

## Introduction

Fast polymerization reactions triggered by UV light occur too quickly to be captured by the human eye or standard cameras.

We present a low-cost embedded system that detects the UV flash and records the reaction using high-speed imaging, enabling slow-motion playback for detailed observation and live demonstration.
## System Architecture

1. A UV flash occurs, triggering a fast polymerization reaction.
2. An analog UV sensor detects the flash.
3. The signal is digitized by an external ADC and read by a Raspberry Pi 3.
4. Upon detection, the Raspberry Pi captures a high-speed burst of frames using the Raspberry Pi Camera Module v2.
5. Frames are buffered in RAM for fast access.
6. Playback is displayed on an HDMI screen and controlled via physical buttons (Play / Pause / Reset).
## System Components

- **Raspberry Pi 5** – Responsible for capturing images and displaying playback.
- **Raspberry Pi Camera Module v2** – Used to record high-speed frames of the reaction.
- **GUVA-S12SD UV Sensor** – Detects the presence of UV light to signal the start of recording.
- **External ADC** – Converts the analog signal from the UV sensor to a digital format.
- **HDMI Display** – Displays the slow-motion playback of the recorded footage.
- **Physical Buttons (Play / Pause / Reset)** – Used for user interaction with playback.
  
## Optimizing Capture Speed

To successfully capture the fast polymerization reaction, the system must record a short, intense event that completes in under 100 milliseconds. Standard camera configurations are inadequate for such timing, so we needed to significantly increase the camera’s frame rate.

The Raspberry Pi Camera Module v2 uses a **rolling shutter**, meaning it reads each frame line by line (row by row) from top to bottom, rather than capturing the entire image at once. As a result, the time it takes to capture a full frame is directly proportional to the number of rows. By reducing the **vertical resolution** (height) of the image, the camera completes each frame readout more quickly allowing for much higher frame rates.

In our configuration, we reduced the image height to only 64 rows using the `-h 64` flag. This trade-off results in a narrow horizontal viewing band, but enables the camera to reach approximately 975 frames per second when combined with increased analog gain and optimized sensor parameters. Reducing the **width** of the image, on the other hand, has negligible impact on performance, since each row is still read as a unit. Therefore, vertical reduction is the most effective method for achieving high-speed capture with this sensor.

The resulting frames were buffered in RAM to ensure minimal write delays, and later processed into a slow-motion video using precise timing information for synchronization.

---

## Frame Processing and Output

We used the open-source `raspiraw` tool to capture raw Bayer-format frames directly from the Raspberry Pi Camera Module v2. Captured frames were converted to `.tiff` format using `dcraw`, and then compiled into a synchronized slow-motion video using `ffmpeg`, referencing the original timestamps.

This pipeline allowed us to reconstruct the short, high-speed event into a temporally stretched video that reveals details not visible at normal playback speeds.

Currently, the recorded frames appear dark due to limited lighting during initial testing. We are experimenting with improved lighting conditions, but expect that the final exhibit setup, which includes a strong UV flash during the reaction, will provide sufficient illumination without the need for supplemental light sources. If needed, we have the option to integrate additional lighting into the exhibit to ensure consistent visibility and recording quality.

---

## System Testing Update

To evaluate the high-speed capture pipeline before testing the actual polymerization reaction, we conducted a preliminary run using a simple scene: a rotating fan under dim indoor lighting.

**[View sample capture](https://drive.google.com/file/d/1PzW4zkaScqAXy1D3jOnbjnEaSPEWVI3p/view)**

This recording confirms that the system can detect and respond to visual motion and brief light changes, even under minimal illumination. Although the frames appear dark, the capture timing and buffering logic performed as expected.

---

## Outdoor Lighting Test

To verify that the camera performs well under stronger lighting conditions, we conducted an outdoor test using daylight and a stream of water as the subject. The goal was to ensure that the system reliably captures fast motion when sufficient illumination is available.

This test reused the same optimized capture configuration, with a vertical resolution of 64 rows and a frame rate of approximately 975 fps. Due to the limited frame height, the visible portion of the scene is reduced to a narrow horizontal band, requiring careful alignment of the subject within this region. In this case, the camera was sometimes rotated or inverted to better align the subject vertically within the reduced frame.

### Sample Captures

1. **[Sample 1 – Stream falling vertically](https://drive.google.com/file/d/1tkDQBM2TiiqNo0x5NmPXNWECEWIrQ5F-/view)**  
   The water stream is clearly visible as droplets fall, although the image appears upside down due to the camera being physically inverted. Despite this, the capture demonstrates that the system successfully records fast vertical motion within the limited frame height.

2. **[Sample 2 – Droplets hitting the ground](https://drive.google.com/file/d/1HDNrdlW91r2VxZYlnDlstKxNU1lgPaSQ/view)**  
   After correcting the camera orientation, the main water stream was missed due to imperfect horizontal alignment within the reduced frame height. However, droplets that rebounded off the ground were still captured, demonstrating motion detection near the frame edges.

3. **[Sample 3 – Less successful attempt](https://drive.google.com/file/d/1XEx5hrPEJuywzcE6ZUDVG6fwlSdQVgLJ/view)**  
   This capture was less successful, but still verifies the system’s ability to detect and record motion. At this stage, we chose not to further fine-tune alignment, as the results were sufficient to validate the functionality ahead of exhibit-specific testing.

---

## Exhibit Testing – First On-Site Capture Attempts

After validating the system under outdoor conditions, we conducted the first test on the actual exhibit. The goals of this session were to evaluate whether the built-in UV flash provides sufficient illumination, determine an initial camera position that captures the polymerization droplet clearly, and select a frame rate that balances motion resolution and recording duration.


**Preliminary attempt – Width 480 (failed)**  
We initially attempted to capture at width 480 and height 64 for 2 seconds at 660 fps  
The system returned a memory overflow error, likely due to buffer limitations at this width and frame rate combination  
No video was recorded  

**Attempt 1 – 'first_angle' [video link](https://drive.google.com/file/d/1XnBNxm8tU3VQpRA91qte8zRlW4S25yey/view?usp=sharing)**  
Camera placed behind the exhibit at very short distance  
Height 64, width 320, 660 fps, duration 1.5 seconds  
Result: extreme overexposure due to UV flash  
Saturation begins around 0.05 seconds and persists for ~5 seconds, obscuring the droplet completely  

**Attempt 2 – 'second_angle' [video link](https://drive.google.com/file/d/1Hyd2K05YBTS2U5h5Ollt0MIzkuETlodg/view?usp=drive_link)**  
Camera repositioned to doorway, approximately 0.5 meters from the exhibit  
Same resolution and duration as above  
The UV flash still causes significant overexposure around 0.20 seconds  
Droplet motion is not clearly visible  

**Attempt 3 – 'position_cam' [video link](https://drive.google.com/file/d/1Hyd2K05YBTS2U5h5Ollt0MIzkuETlodg/view?usp=drive_link)**  
Switched to 90 fps to help determine spatial framing  
The droplet is partially visible and suggests the need to shift the camera slightly to the left and downward  

**Attempt 4 – 'position_cam_2ndTry' [video link](https://drive.google.com/file/d/1h7z4Tb6kj34zj2ObndDjuNtJaOW4lgTu/view?usp=drive_link)**  
Adjustment made based on previous framing  
Droplet visibility improved, but further refinement is needed  

**Attempt 5 – 'position_cam_3rdTry' [video link](https://drive.google.com/file/d/1QLZG5moAc6I2PqqHCZD2S2QvedE6nByG/view?usp=drive_link)**  
Minor additional adjustments  
Closer to correct framing but still slightly misaligned  

**'heigh-64--fps-660' [video link](https://drive.google.com/file/d/1kSxTVTmyV0xH4dVVgWqpBf7O7T3rRGqm/view?usp=drive_link)**  
Camera placement appears to be close to optimal  
Droplet is more consistently visible and the UV flash no longer overwhelms the image  
This test served as a reference point for further adjustments 

**Final test of the day – 'high-fps-final' [video link](https://drive.google.com/file/d/1PDiuaL70WirRHc-hoIIKpiJEuty05d2K/view?usp=drive_link)**  
Same camera position as in the 660 fps test, but captured at 1000 fps to improve temporal resolution  
Motion appears significantly smoother, with the droplet becoming visible around 0.07 seconds  
Compared to the 660 fps version, this recording shows less choppiness and better continuity of motion.

---

## **Mouse Based Triggering of Video Capture**

After validating the high-speed capture pipeline and obtaining promising initial results, we added functionality to trigger the recording process via a mouse click. This change streamlines operation and makes the system more suitable for interactive use during testing and future exhibit deployment.

To implement this, we connected a USB mouse to the Raspberry Pi 3 and installed the necessary dependencies using apt, including evtest, python3-evdev, and ffmpeg. We then used the ls /dev/input/by-id/ command to identify the device name associated with the mouse, and confirmed the specific event path was event1. We verified this using the evtest command, which displayed real-time input events. For example, when pressing the right mouse button, we observed:

Event: time 1753012817.775234, type 4 (EV_MSC), code 4 (MSC_SCAN), value 9000  
Event: time 1753012817.775234, type 1 (EV_KEY), code 273 (BTN_RIGHT), value 1  
Event: time 1753012817.775234, type 0 (EV_SYN), code 0 (SYN_REPORT), value 0  

Based on this, we wrote a Python script named mouse_trigger.py that listens for right or middle click events and launches the video capture pipeline accordingly. We made the script executable using chmod +x mouse_trigger.py, tested it with sudo ./mouse_trigger.py, and verified that it successfully triggers the recording process when the mouse is clicked. While we intend to eventually configure the script to run automatically on boot using systemd, we have postponed this step for now in order to focus on other development priorities. The script is included in the scripts/ directory of this repository.

---

## Prototype Assembly and Function

The current stage of the project is focused on making the existing prototype of the museum exhibit fully operational so it can serve as a testbed for the development of the high-speed camera system. This prototype, which was already built for earlier photopolymer drop demonstrations, reproduces the key functional principles of the final installation at a smaller scale. It allows us to integrate the camera with the detection, triggering, curing, and droplet control mechanisms before moving to the final exhibit build.

The main goal is to ensure that every subsystem works reliably and that they integrate seamlessly into one coordinated process, from dispensing a photopolymer drop, to detecting its presence, to triggering the UV curing light precisely at the correct moment.

The system consists of four main subsystems:

1.  **Drop Detection System**
    The drop detection unit is a custom-built sensor designed and assembled in house at the museum specifically for this project. Its design is based on the provided schematic and uses a dedicated light source and photodetector arranged so that a passing droplet partially blocks or scatters the beam. This creates a measurable change in the sensor output, which is processed by conditioning electronics that filter noise, stabilize the signal, and generate a clean digital trigger.

  The detection module is implemented on a compact, project specific PCB that includes the photodetector, connectors, and all necessary passive components for stable and repeatable operation. It interfaces directly with the control shield for real time triggering of the curing light.

<figure>
  <img src="Schematic_Drop_sensor_2025-07-21.png" width="500"/>
  <figcaption>Schematic, Drop Sensor</figcaption>
</figure>

<figure>
  <img src="PCB_PCB_Drop_sensor_2_2025-07-21.png" width="500"/>
  <figcaption>PCB Layout, Drop Sensor</figcaption>
</figure>

2. **UV Curing Light System**
   
  Once a droplet is detected, the system must trigger the curing light with minimal delay. The UV curing subsystem consists of a custom high power UV LED module, also designed and assembled in house. The LED array is driven by a dedicated circuit that provides a stable current and precise timing control.

  The curing wavelength is matched to the photopolymer’s requirements, ensuring rapid solidification. Pulse duration is carefully controlled,  too short and the curing will be incomplete, too long and it could negatively affect the visual quality of the demonstration.

<figure>
  <img src="Schematic_PHOTOPOLYMER_CURE_LIGHT_V0_0_2025.png" width="500"/>
  <figcaption>Schematic, UV Curing Light</figcaption>
</figure>

<figure>
  <img src="PCB_PCB_PHOTOPOLYMER_CURE_LIGHT_V0_0_2025-07-21.png" width="500"/>
  <figcaption>PCB Layout, UV Curing Light</figcaption>
</figure>

3. **Control and Integration Shield**

   At the core of the system is a custom control shield that mounts on an Arduino based microcontroller. This shield acts as the central hub for all electrical connections, signal routing, and power management.

  It receives the digital trigger from the drop detection system, sends the activation signal to the UV curing light, and controls the stepper motor driver for droplet dispensing. It also allows for flexible configuration during testing, enabling quick changes to timing parameters and operational modes.

<figure>
  <img src="Schematic_Shield_Amir_ELAD_V_0_0_2023-11-08.png" width="500"/>
  <figcaption>Schematic, Control Shield</figcaption>
</figure>

<figure>
  <img src="PCB_PCB_Shield_Amir_ELAD_V_0_0_2025-07-21.png" width="500"/>
  <figcaption>PCB Layout, Control Shield</figcaption>
</figure>

4. **Stepper Motor and Peristaltic Pump**
   The droplet dispensing mechanism is powered by a bipolar stepper motor coupled to a peristaltic pump. This setup ensures precise droplet volume control and consistent release intervals, both of which are essential for accurate triggering and curing.

  The stepper motor is driven by a dedicated driver controlled by the shield, allowing the firmware to adjust speed and timing parameters. Mechanical alignment of the nozzle relative to the detection beam is critical, even small deviations can affect detection accuracy.



## **Arduino to Raspberry Pi Trigger Interface**


To ensure precise control over the start of high-speed video capture, we are planning a direct UART connection between the Arduino and the Raspberry Pi using GPIO pins. The Arduino detects the UV flash via an analog sensor and immediately sends a serial trigger to the Raspberry Pi to initiate recording.
Details about the sensor will be provided later.

Because the Arduino operates at 5V logic while the Raspberry Pi’s GPIO pins are limited to 3.3V, we have incorporated a passive voltage divider to safely reduce the voltage level on the Arduino’s TX line before it reaches the Pi’s RX pin.

The schematic of the voltage divider is shown below:

<img src="voltage_divider_schematic.png" width="500"/>
We selected two standard resistor values: 1.1 kΩ for R₁ and 2 kΩ for R₂, forming a simple voltage divider, which yields the following output voltage:


<img src="voltage_divider_formula.png" width="300"/>
This voltage is within the safe input range for the Raspberry Pi GPIO and is well above the minimum voltage required to register a logical HIGH.
According to the BCM2835 datasheet and technical documentation, the threshold for detecting a HIGH input is:

<img src="vih_threshold.png" width="300"/>
Thus, a signal of approximately 3.22 V provides a safety margin of over 0.9 V above the threshold, ensuring reliable logic level detection.


## Next Steps

??
- Improve lighting setup or confirm sufficiency of UV flash in exhibit conditions
- Test the system with the actual museum hardware and environment
- Add playback control logic (Play / Pause / Reset) using physical buttons
- Evaluate different frame processing pipelines or export options
- Package the system in a compact, exhibition-ready form

