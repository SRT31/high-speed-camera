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

To successfully capture the fast polymerization reaction, the system must record a short, intense event that completes in under 100 milliseconds. Standard camera configurations are inadequate for such timing, so we needed to significantly increase the camera’s frame rate.

The Raspberry Pi Camera Module v2 uses a **rolling shutter**, meaning it reads each frame line by line (row by row) from top to bottom, rather than capturing the entire image at once. As a result, the time it takes to capture a full frame is directly proportional to the number of rows. By reducing the **vertical resolution** (height) of the image, the camera completes each frame readout more quickly — allowing for much higher frame rates.

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

This test reused the same optimized capture configuration, with a vertical resolution of 64 rows and a frame rate of approximately 660 fps. Due to the limited frame height, the visible portion of the scene is reduced to a narrow horizontal band, requiring careful alignment of the subject within this region. In this case, the camera was sometimes rotated or inverted to better align the subject vertically within the reduced frame.

### Sample Captures

1. **[Sample 1 – Stream falling vertically](https://drive.google.com/file/d/1tkDQBM2TiiqNo0x5NmPXNWECEWIrQ5F-/view)**  
   The water stream is clearly visible as droplets fall, although the image appears upside down due to the camera being physically inverted. Despite this, the capture demonstrates that the system successfully records fast vertical motion within the limited frame height.

2. **[Sample 2 – Droplets hitting the ground](https://drive.google.com/file/d/1HDNrdlW91r2VxZYlnDlstKxNU1lgPaSQ/view)**  
   After correcting the camera orientation, the main water stream was missed due to imperfect horizontal alignment within the reduced frame height. However, droplets that rebounded off the ground were still captured, demonstrating motion detection near the frame edges.

3. **[Sample 3 – Less successful attempt](https://drive.google.com/file/d/1XEx5hrPEJuywzcE6ZUDVG6fwlSdQVgLJ/view)**  
   This capture was less successful, but still verifies the system’s ability to detect and record motion. At this stage, we chose not to further fine-tune alignment, as the results were sufficient to validate the functionality ahead of exhibit-specific testing.


## Next Steps

??
- Improve lighting setup or confirm sufficiency of UV flash in exhibit conditions
- Test the system with the actual museum hardware and environment
- Add playback control logic (Play / Pause / Reset) using physical buttons
- Evaluate different frame processing pipelines or export options
- Package the system in a compact, exhibition-ready form

