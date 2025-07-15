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

---

### Outdoor Lighting Test

To verify that the camera performs well under stronger lighting conditions, we conducted an outdoor test using daylight and a stream of water as the subject. The goal was to ensure that the system reliably captures fast motion when illumination is sufficient.

To achieve high frame rates (around 660 fps), we reduced the vertical resolution to only 64 rows using the `-h 64` flag. This significantly increases the frame rate, as the Raspberry Pi Camera Module v2 uses a **rolling shutter**, which reads out the image **row by row**, rather than capturing the whole frame at once. Reducing the number of **rows** directly reduces the readout time, enabling higher capture speeds. In contrast, reducing the **width** (number of columns per row) has much less effect on performance. As a result, rotating the camera 90 degrees and reducing the height is a common technique to capture fast vertical motion like falling water.

While this configuration improves timing, it limits the visible portion of the scene to a narrow horizontal (or vertical, if rotated) band' similar to looking through a thin slit. This trade-off requires careful physical alignment to ensure that the subject is within the visible area.

#### Sample Captures

1. **[Sample 1 – Stream falling vertically](https://drive.google.com/file/d/1tkDQBM2TiiqNo0x5NmPXNWECEWIrQ5F-/view)**  
   The water stream is clearly visible as droplets fall, although the image appears upside down due to the camera being physically inverted. Despite this, the capture demonstrates that even in this orientation, the system successfully records fast vertical motion within the limited frame height.

2. **[Sample 2 – Droplets hitting the ground](https://drive.google.com/file/d/1HDNrdlW91r2VxZYlnDlstKxNU1lgPaSQ/view)**  
   After correcting the camera orientation, the main water stream was missed due to imperfect horizontal alignment within the reduced frame height. However, droplets that rebounded off the ground were still captured, demonstrating motion detection even near the edges of the frame.

3. **[Sample 3 – Less successful attempt](https://drive.google.com/file/d/1XEx5hrPEJuywzcE6ZUDVG6fwlSdQVgLJ/view)**  
   This capture was less successful, but still demonstrates the system’s ability to detect fast motion. At this stage, we chose not to focus on further precision, as the results are sufficient to validate overall functionality before tuning the system for the exhibit setup.




## Next Steps

??
- Improve lighting setup or confirm sufficiency of UV flash in exhibit conditions
- Test the system with the actual museum hardware and environment
- Add playback control logic (Play / Pause / Reset) using physical buttons
- Evaluate different frame processing pipelines or export options
- Package the system in a compact, exhibition-ready form

