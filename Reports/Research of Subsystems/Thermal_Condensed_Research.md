# Thermal Subsystem Condensed Research

**Valinor Dispatch drone inspection  | October 22, 2026**

## Purpose

My part of the project is to use a thermal camera to check for unusual heating while the drone is parked in the dock. I plan to start with visible areas such as motor housings, the battery casing, and charging connections. These measurements will give the inspection system information that the regular cameras cannot provide on their own.

For the first prototype, I recommend one FLIR Lepton 3.5 connected through a PureThermal 3 USB board. The dock already uses a Jetson Orin Nano Super 8GB, so the plan is to process the thermal images there alongside the other camera inputs. [1–3]

## Camera and software choices

The Lepton 3.5 has a resolution of 160 × 120 pixels, a 57° horizontal field of view, and a frame rate of about 8.6 frames per second. It can provide temperature readings for each pixel, which makes it worth testing for this application. Whether it provides enough detail will depend on the size of the component and how close we can mount it. [2]

I would use the PureThermal 3 because it lets the Jetson receive the images over USB. A basic breakout board would cost less, but we would have to spend more time setting up SPI capture and handling errors in the video stream. For this first version, I would rather put that effort into checking the measurements and getting useful results from the camera. [3,5]

The thermal software can start with OpenCV and NumPy. It will read the original 16-bit temperature data and measure selected areas of the image. The colored thermal image will be useful for reviewing results, but the temperature checks need to use the underlying data. [4]

## How it fits into the team project

The thermal camera will work alongside the UVC cameras and the DeepStream/GStreamer and YOLOv4 inspection software. The visible-camera system will identify drone components, while my subsystem will provide temperature measurements for the areas we can reliably match to those components. To get started, I can use fixed regions in the thermal image before connecting them to the component detections.

The planned inspection sequence is:

1. The dock confirms that the drone is secured.
2. The thermal camera takes its readings with the visible inspection lights initially off.
3. The lights turn on for the regular camera inspection.
4. The software combines the findings for each component and saves the images and results.

I will send the AI/ML portion the component ID, temperature readings, capture time, and any reason the reading could not be used. That will support the overall PASS, WARN, FAIL, or UNVERIFIED result. Charging and flight decisions will remain with the dock controller.

I will also need to work with the mounting and power portions of the project to choose a camera position and route the USB cable. We should test whether heat from the lights affects later readings. If an acoustic test involves running the motors, we will need to account for that heating in the inspection sequence.

## Placement and temperature checks

Valinor lists the dock dimensions as 30 × 12 × 40 inches, but we still need to measure the space available around the drone. [1] As a rough calculation, the Lepton's view at a 12-inch distance would cover about 13 × 10 inches on a flat surface facing the camera. One camera may therefore cover only part of the drone.

Before setting alarm limits, I need to compare the camera readings with a suitable temperature reference. Reflections and differences in surface material can affect the result. The camera also measures the visible surface, so a reading from the battery casing does not directly tell us the temperature inside the cells. [4]

Most of the useful data will come from our own dock tests. I plan to record normal conditions and use controlled heated targets to test whether the software notices a temperature increase. Readings after flight should be treated separately from readings during charging. I will use separate test sessions to check whether the rules work beyond the recordings used to develop them.

## Cost estimates

| Item | Rough cost |
|---|---:|
| Lepton 3.5 | $170–180 |
| PureThermal 3 | About $120 |
| Powered USB hub, if needed | $50–80 |
| Total for one camera setup | About $330–440 |

The camera and board estimates come from GroupGets listings checked October 20, 2026. The remaining amounts are allowances. Tax, shipping, the existing Jetson, and purchased calibration equipment are not included. [2,3]

## Testing and standards

The first goal is to show that the camera gives repeatable readings and detects a controlled temperature increase in the intended area. After that, I will check what happens when the view is blocked, the camera disconnects, or the other sensors are running. An unavailable or outdated reading should be marked UNVERIFIED. We will need to agree on acceptable temperature error and inspection time before deciding that the subsystem is ready.

The most relevant standards to consult are:

- **ISO 18434-1 and ISO 18434-2:** planning thermal inspections and interpreting the images. [6]
- **ASTM E1933 and E1934:** accounting for surface emissivity and inspecting electrical and mechanical equipment. [7]
- **ISO/IEC/IEEE 29148 and IEEE 1012:** writing testable requirements and planning how to verify them. [8]

These will help guide the design and testing. Which requirements we must formally meet still depends on the sponsor and the final project scope.

## References

[1] Valinor, [Dispatch specifications](https://www.valinor.co/products/dispatch).

[2] Teledyne FLIR, [Lepton specifications](https://oem.flir.com/products/lepton/); GroupGets, [Lepton 3.5 purchase listing](https://groupgets.com/products/flir-lepton-3-5).

[3] GroupGets, [PureThermal 3 specifications and purchase listing](https://groupgets.com/products/purethermal-3).

[4] Teledyne FLIR, [Radiometry with Lepton](https://oem.flir.com/learn/thermal-integration-made-easy/radiometry-with-lepton/); GroupGets, [UVC capture and radiometry examples](https://github.com/groupgets/purethermal1-uvc-capture).

[5] GroupGets, [PureThermal Breakout Board](https://groupgets.com/products/purethermal-breakout-board).

[6] ISO, [18434-1:2008](https://www.iso.org/standard/41648.html) and [18434-2:2019](https://www.iso.org/standard/67617.html).

[7] ASTM, [E1933-14(2022)](https://store.astm.org/e1933-14r22.html) and [E1934-99a(2024)](https://store.astm.org/e1934-99ar24.html).

[8] ISO, [ISO/IEC/IEEE 29148:2018](https://www.iso.org/standard/72089.html); IEEE, [1012-2024](https://standards.ieee.org/ieee/1012/7324/).
