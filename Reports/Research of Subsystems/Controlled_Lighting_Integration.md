# Controlled Lighting Integration

## General Overview

### Lighting Choice

I propose white LED lighting with removable diffusers for the visible-camera inspection of the docked drone. The purpose is to create repeatable imaging conditions while limiting shadows, hotspots, and reflections that can hide surface details. My starting arrangement uses adjustable light positions on opposing sides of an inspection region. I will compare this general illumination with shallow-angle directional lighting and retain an additional mode if it improves defect visibility. [Lighting]

A single white-light spectrum will be used within each mode. Neutral or cool white is my initial preference, with warm white available for comparison if surface colors affect the images. The final choice will depend on contrast, glare, and consistency rather than assuming that a particular color temperature is better for detecting damage.

### Functionality and Interfacing

The lights will operate during image acquisition and remain off between inspection cycles. For the proposed strip arrangement, the lighting load will receive external 12 V DC power. The NVIDIA Jetson Nano will provide control signals through compatible switching hardware. Its GPIO will not carry the LED load current. The original Nano uses a 5 V board supply and 3.3 V interface signals. [Nano]

My scope covers the light sources, diffusers, mounting arrangement, brightness, and lighting operation. The supply voltage and control connections define how this subsystem connects to the overall electrical and camera design.

## Physical Layout Plan

### Design Basis and Constraints

For this concept, I am using the horizontal dimensions in our camera integration draft as the working design basis. [Team layout]

| Parameter | Working design basis |
|---|---|
| Dock enclosure footprint | 30 in wide by 40 in long |
| Drone footprint | 20 in wide by 20 in long, centered in the enclosure |
| Inspection condition | Stationary drone during visible capture |
| Feature target | 0.5 mm, carried over as a preliminary inspection goal from the camera draft |

The 0.5 mm target will be included in lighting trials using representative samples. Successful detection will also depend on camera resolution, focus, viewing angle, and the detection method; lighting alone does not establish that capability.

### General Measurements and Clearances

For a centered drone, the horizontal clearance on each side is half the difference between the enclosure span and the drone span:

$$
C = \frac{D_{\mathrm{enclosure}} - D_{\mathrm{drone}}}{2}
$$

| Direction | Calculation | Nominal clearance on each side |
|---|---|---|
| Across the 30 in enclosure width | (30 - 20) / 2 | 5 in, or 127 mm |
| Across the 40 in enclosure length | (40 - 20) / 2 | 10 in, or 254 mm |

These values describe wall-to-footprint clearances. Brackets, diffuser thickness, the landing position, and the camera arrangement affect the usable space and light-to-surface distance.

### Lighting Arrangement Evaluation

#### Arrangement 1 Opposed Diffused Lighting

My baseline is two opposed light positions illuminating one inspection region. Each source will have a removable diffuser and an adjustable bracket so I can compare positions without committing to a fixed enclosure design. The lights will be positioned clear of the corner camera locations proposed in the camera draft and outside the docking path. The initial bench arrangement uses two 1 m strips.

Diffusion will be evaluated for reducing hotspots and hard shadows. Its size and spacing matter: a narrow strip behind a diffuser does not automatically provide uniform illumination over the whole drone. I will check actual propellers, arms, housings, and hardware rather than relying only on a flat reference target. [Lighting]

#### Arrangement 2 Directional Defect Lighting

I will compare shallow-angle illumination from opposing directions to evaluate the visibility of chips, scratches, edges, and representative cracks. Defects can respond differently to the direction of illumination. Bench fixtures can be repositioned during testing; an automated implementation would need repeatable fixed channels or another positioning method.

This mode will be retained if the images show a useful improvement. I will use the term dark-field only where the geometry directs ordinary reflections away from the camera while useful scattered light reaches it. A ring or dome arrangement remains an alternative if the bar-light approach cannot provide adequate coverage or glare control. [Lighting]

## Lighting Recommendation

### Candidate Hardware

I recommend low-cost white LED strips for the first mounting and diffusion trials.

| Candidate | Main published details | Listed price | Intended role |
|---|---|---|---|
| Adafruit 4612 cool-white COB strip | 1 m; nominal 12 V; 10 W/m; 6000-6500 K | $11.95 | Initial general-lighting prototype [Cool strip] |
| Adafruit 4613 warm-white COB strip | 1 m; approximately 3000 K; supplier describes 9-12 V operation | $11.95 | Optional spectrum comparison [Warm strip] |

The listed strip prices were checked on October 20, 2026, before tax and shipping.

### Preliminary Strip Parameters and Power Budget

The following values apply to the Adafruit 4612 cool-white candidate. Published output is a source specification and does not establish illuminance on the drone after diffusion. [Cool strip]

| Parameter | Published or calculated value |
|---|---|
| Operating voltage | 12 V DC nominal |
| Power per metre | 10 W/m |
| Current per metre | Approximately 0.83 A at 12 V, calculated |
| Color temperature | 6000-6500 K |
| LED density | 480 LEDs/m |
| Published light output | 1100 lm/m |
| Published beam angle | 180 degrees |
| Nominal PCB dimensions | 1000 mm long by 10 mm wide |

For the initial two-strip arrangement:

$$
P_{\mathrm{total}} = 2 \times 10\,\mathrm{W} = 20\,\mathrm{W}
$$

$$
I_{\mathrm{total}} = \frac{P_{\mathrm{total}}}{V} = \frac{20\,\mathrm{W}}{12\,\mathrm{V}} \approx 1.67\,\mathrm{A}
$$

I propose a regulated 12 V, 3 A lighting supply, providing 36 W of rated capacity for the nominal 20 W initial load. I will verify actual current, voltage stability, and operating temperature during bench testing. These power calculations apply to the cool-white strips. [Cool strip]

### Lighting Components and Cost

The proposed assembly includes LED sources, removable diffusers, adjustable brackets, compatible switching channels, and the necessary lighting power connections. Polarizing filters remain optional if diffusion does not sufficiently reduce reflections.

| Item | Qty | Unit price | Line total |
|---|---|---|---|
| Adafruit 4612 cool-white strip, 1 m | 2 | $11.95 | $23.90 |
| Adafruit 5648 switching board | 2 | $3.95 | $7.90 |
| Base subtotal for sources and switches | - | - | $31.80 |
| Optional Adafruit 4613 warm-white strip, 1 m | 1 | $11.95 | $11.95 |
| Subtotal with optional comparison strip | - | - | $43.75 |

These are component subtotals for the listed light sources and switches. They exclude tax, shipping, the lighting supply or converter, diffusers, brackets, protection, connectors, and any additional directional fixtures. [Cool strip; Warm strip; Switch]

## Control and Compatibility

The proposed controller is the NVIDIA Jetson Nano. Its GPIO interface will command the lighting switches, with one independently controlled channel per 1 m strip in the initial arrangement. The candidate Adafruit 5648 board accepts 3-20 V logic and is rated for 1.5 A continuous load current. One 1 m strip has a calculated load of about 0.83 A; both full strips together exceed the continuous rating of one board. Compatibility and temperature will still be checked in the assembled system. [Nano; Switch]

Steady illumination throughout each visible capture is the initial operating method. Any PWM dimming or short-pulse operation will be evaluated with the selected cameras because rolling-shutter rows expose at different times. A software command alone does not confirm that the intended illumination reached the scene. [Camera timing]

## Connections

- **Lighting power:** A regulated 12 V source can supply the strips through the switches. A higher source, such as 24 V, needs a suitable buck converter with a proposed 12 V, 3 A continuous output. A 5 V source needs a boost converter; a source crossing 12 V needs buck-boost conversion. My initial bench setup uses a separate regulated 12 V supply, so no buck converter is needed for that setup.
- **Jetson control:** The Nano sends 3.3 V commands to compatible switches, with a defined common ground for the non-isolated arrangement. The 12 V lighting rail will not feed the Nano's 5 V input or GPIO. The strips draw their load current from the external lighting supply. [Nano; Switch]
- **Physical mounting:** Adjustable brackets hold the chosen angles and distances. Removable diffusers support testing and cleaning while preserving camera views and docking clearance.
- **Camera coordination:** Capture follows lighting stabilization. Buffered frames from the previous mode will be rejected. Accepted images will carry the lighting mode and capture time.

## Operation and Verification

### Proposed Inspection Sequence

1. Visible lights remain off for the proposed initial thermal capture.
2. General lighting turns on and stabilizes for the visible inspection views.
3. Directional lighting provides additional views if that mode is retained.
4. All inspection lights turn off after capture.

I will evaluate this sequence over repeated cycles. Fixtures can retain heat after switching off, and warm surroundings can affect thermal reflections from some surfaces. Thermal-first capture is therefore a proposed sequence to test, rather than an assurance that the lighting has no thermal influence. [Thermal reflections]

### Lighting Verification

| Check | Evidence I will collect |
|---|---|
| Coverage and uniformity | Reference-target images at relevant positions and actual component views |
| Glare and defect contrast | Images of representative finishes and known defect sizes under each mode |
| Repeatability | Reference-region brightness over repeated cycles with camera settings fixed |
| Capture compatibility | Frames checked for banding, brightness variation, and mode transitions |
| Thermal interaction | Matched lights-off and operating-cycle trials, including residual fixture heat |
| Lighting faults | Verification of the selected mode, the off state, and missing-light detection |

## References

- **Team layout:** Project team, UVC Camera Integration, supplied conceptual design draft, 2026.
- **Lighting:** Edmund Optics, [Common Illumination Types](https://www.edmundoptics.com/knowledge-center/application-notes/illumination/choose-the-correct-illumination/).
- **Cool strip:** Adafruit, [Ultra Flexible White LED Strip, product 4612](https://www.adafruit.com/product/4612), and the linked [iPixel LED datasheet](https://cdn-shop.adafruit.com/product-files/4612/C15482-001__6500K_.pdf).
- **Warm strip:** Adafruit, [Ultra Flexible White LED Strip, product 4613](https://www.adafruit.com/product/4613).
- **Nano:** NVIDIA, [Jetson Nano Developer Kit User Guide](https://developer.download.nvidia.com/embedded/L4T/r32-3-1_Release_v1.0/Jetson_Nano_Developer_Kit_User_Guide.pdf), December 2019.
- **Switch:** Adafruit, [MOSFET Driver, product 5648](https://www.adafruit.com/product/5648).
- **Camera timing:** Basler, [Electronic Shutter Types](https://docs.baslerweb.com/electronic-shutter-types).
- **Thermal reflections:** FLIR, [How Does Emissivity Affect Thermal Imaging](https://www.flir.com/en-asia/discover/professional-tools/how-does-emissivity-affect-thermal-imaging/).
