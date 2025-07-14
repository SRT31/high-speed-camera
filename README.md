# High-Speed Camera System
An embedded system for visualizing fast UV-triggered polymerization reactions in slow motion.
## Introduction

Fast polymerization reactions triggered by UV light occur too quickly to be captured by the human eye or standard cameras.

We present a low-cost embedded system that detects the UV flash and records the reaction using high-speed imaging, enabling slow-motion playback for detailed observation and live demonstration.
## System Architecture

1. A UV flash occurs, triggering a fast polymerization reaction.
2. An analog UV sensor (GUVA-S12SD) detects the flash.
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

To record the fast polymerization reaction, we had to increase the camera's frame rate.  
Since the Raspberry Pi Camera Module v2 has a rolling shutter and limited frame rate at full resolution, we minimized the image height to only 64 rows.  
By reducing the height and increasing the analog gain, we achieved a frame rate of approximately 975 fps.  
This allowed us to capture the entire UV-triggered reaction within a 100ms burst.

The captured frames were then stored in RAM and later processed into a slow motion video using frame by frame timing.
## Frame Processing and Output

We used the open-source `raspiraw` tool to capture raw Bayer frames directly from the Raspberry Pi Camera Module v2.  
By minimizing the image height and optimizing sensor parameters, we achieved a frame rate of approximately **975 fps**.

Captured frames were converted to `.tiff` format using `dcraw`, and then compiled into a synchronized slow-motion video using `ffmpeg`, based on the original timestamp data.

Currently, the recorded frames appear **too dark** due to limited lighting during testing.  
We are experimenting with different lighting conditions,  
but in the **actual exhibit**, a strong UV flash is present during the reaction — so additional lighting may **not** be necessary in the final setup.

---

### System Testing Update

To evaluate the high-speed capture pipeline before testing the actual polymerization reaction, we conducted a preliminary run using a simple scene  a rotating fan under dim lighting conditions.

**[View sample capture](https://drive.google.com/file/d/1PzW4zkaScqAXy1D3jOnbjnEaSPEWVI3p/view)**

This recording confirms that the system can detect and respond to visual motion and brief light changes, even under minimal illumination. Although the frames appear dark, the capture timing and buffering logic worked as expected.


## Next Steps

??
- Improve lighting setup or confirm sufficiency of UV flash in exhibit conditions
- Test the system with the actual museum hardware and environment
- Add playback control logic (Play / Pause / Reset) using physical buttons
- Evaluate different frame processing pipelines or export options
- Package the system in a compact, exhibition-ready form

