# Conceptual Design — Integrated In-Dock Drone Inspection System

**Valinor Dispatch: Multisensor, AI/ML-Assisted Drone Inspection**

**Team 06 | Selectively revised conceptual design | November 18, 2026**

**Document status:** Proposed engineering design for team and client review. Unless a requirement is identified as a supplied physical constraint or published device specification, numerical targets and design choices are preliminary and have **not** been verified by testing or approved by Valinor. This document combines the team's RGB camera, controlled lighting, thermal, acoustic, and ML research in the formatting style of the original ML conceptual design. [3][5][7][9][11][19]

## 1. Introduction and fully formulated problem

Valinor's Dispatch dock houses a drone between flights. A drone may return with a missing or damaged propeller, surface damage, loose or missing hardware, or an unusually warm component. A conventional camera cannot directly measure surface temperature, and a thermal camera cannot resolve all of the small visible features. The proposed project adds a coordinated inspection system inside the dock to gather visible and thermal measurements, evaluate the components that can actually be observed, and provide a documented maintenance-oriented result. An acoustic measurement path will be explored separately if a safe diagnostic procedure can be approved. [3][5][7][9][11][19] [46]

The selected concept uses four corner-mounted USB Video Class (UVC) cameras, controllable white LED lighting, one FLIR Lepton 3.5 with a PureThermal 3 interface, and the existing NVIDIA Jetson Orin NX 16 GB computing module, with a USB microphone as a conditional extension. The primary processing order is dock authorization → thermal capture with visible lights off → controlled-light RGB capture → image-quality checks → component and thermal measurements → decision-level fusion → report and log. The dock controller, rather than the AI model, remains responsible for flight permission, charging controls, and safety interlocks. [3][5][7][9][11][19]

This is an inspection aid, not an aircraft airworthiness certification device. A component outside the cameras' field of view, an untested defect type, or unreliable sensor data must be reported as an inspection limitation; a model's confidence score does not prove that a drone is safe to fly. [3][9][1]

### 1.1 Fully formulated problem and proposed system requirements

**Problem statement:** The team shall design, prototype, and evaluate an integrated dock inspection system that gathers repeatable RGB and thermal measurements while the drone is safely stationary, identifies designated exterior components, and evaluates observable conditions without compromising normal dock operations. The baseline shall determine whether required components are visible, flag a potentially missing propeller or other required part only when its expected location can be evaluated reliably, and assess unusual relative surface temperatures for mapped thermal regions when calibrated data are available. Chipped propeller edges, cracks, and other fine surface damage are candidate research targets but shall not be claimed as detected until labeled examples and held-out testing support that capability. The system shall provide traceable `PASS`, `WARN`, `FAIL`, or `UNVERIFIED` results to the existing dock software and shall not treat an unobserved or unvalidated condition as evidence of a safe aircraft.

| ID | System-level **shall** statement | Origin and status |
|---|---|---|
| <span style="white-space: nowrap;">SYS-01</span> | The system **shall** begin an inspection only after the dock controller confirms a secured, non-operating drone and an inspection-permitted state. | Safety/operational constraint; proposed, client confirmation needed [3][9][5][6][13] |
| SYS-02 | The system **shall** obtain RGB views of every designated inspectable exterior component using the installed camera arrangement and shall identify any required component that cannot be observed. | Camera and ML team concept; final coverage list TBD [3][9][11][32] |
| SYS-03 | The system **shall** acquire usable thermal measurements from designated visible surface regions before reporting their thermal status. | Thermal team concept; applicable regions depend on validated geometry [5][6][13] |
| SYS-04 | The system **shall** coordinate lighting and capture so accepted RGB images are taken under a known lighting mode with stable exposure. | Lighting/camera repeatability [7][12][11][32] |
| SYS-05 | The system **shall** run the primary inspection without requiring continuous internet access. | Edge-processing design objective [3][9] |
| SYS-06 | The system **shall** provide per-component and overall `PASS`, `WARN`, `FAIL`, or `UNVERIFIED` results with machine-readable reason codes. | Pilots/operators and maintenance technicians need interpretable, actionable results; proposed, Valinor approval pending |
| SYS-07 | The system **shall not** issue an overall `PASS` if any mandatory check is invalid, missing, or unverified. | Pilots and people near flight operations need conservative reporting of uncertain conditions; proposed safety requirement |
| SYS-08 | The system **shall** return a timestamp, unique inspection identifier, component identifiers, data-quality flags, and software/configuration versions through a documented local interface. | Valinor engineering and maintenance need traceable integration records; interface approval pending |
| SYS-09 | The system **shall** preserve diagnostic evidence within an agreed retention and storage limit and shall handle storage or communication failures without silently reporting `PASS`. | Engineering accountability and system reliability [1][3][9] |
| SYS-10 | The system **shall** keep model inference isolated from flight-control and safety-interlock authority. | Valinor engineering and pilots require separation of inspection and flight authorization; proposed safety requirement |
| SYS-11 | The prototype **shall target** an end-to-end inspection result within **5 minutes** of an accepted request, measured from request acceptance through delivery of a complete result message on the installed dock configuration. | Pilot/operator need for timely decisions; preliminary team target awaiting Valinor approval |
| SYS-12 | The component detector **shall target at least 90% recall** for each required visible component class on held-out dock capture sessions; the test report shall include class counts and false negatives. | Pilot and maintenance need to avoid missed components; preliminary target awaiting validation |
| SYS-13 | The software **shall** be able to complete a 60-minute continuous-operation test without an application crash or interference with dock control; sensor and storage faults shall be injected during testing. | Preliminary reliability target [3][9] |
| SYS-14 | The proposed RGB camera arrangement **shall target at least 4 pixels/mm** at designated critical inspection regions, with actual sampling measured at representative positions after mounting. The team **shall not** infer reliable defect recognition from pixel sampling alone; any minimum defect size must be demonstrated with the chosen camera, lens, lighting, processing pipeline, and held-out test images. | Preliminary 16 MP IMX519 camera/ML feasibility goal; requires camera-team and Valinor review |
| SYS-15 | A microphone-based motor diagnostic **shall** remain disabled until the client and team approve a safe test event, a motor/propeller interlock procedure, and repeatable test data. | Acoustic proposal versus dock safety constraint [19][27][3][9] |
| SYS-16 | The system **shall** maintain a stable reporting interface when sensors or compatible ML models are changed. | Valinor integration and maintenance need backwards-compatible reports; proposed interface requirement |
| SYS-17 | The assembly **shall** fit within verified enclosure and drone clearances, avoid obstructing landing, doors, cooling, charging, and service access, and use correctly protected power rails. | Packaging/electrical safety constraints [7][12]–[11][32] |
| SYS-18 | The team **shall** document applicable component, firmware, library, dataset, and pretrained-weight licenses before transferring the prototype to the sponsor. | Professional and intellectual-property constraint [3][9][11][32] |
| SYS-19 | The inspection pipeline **shall target a seeded-fault false-`PASS` rate of no more than 5%**, reported with fault counts and a one-sided 95% confidence bound on independent held-out trials. | Public-safety and maintenance need to minimize missed faults; proposed criterion awaiting client approval |
| SYS-20 | The evaluation report **shall** separately record component-detection precision, recall, mAP, false negatives by category, inference latency, and total inspection time under documented camera, lighting, dataset, and Jetson operating conditions. | Engineering need for measurable and repeatable performance; proposed verification requirement |
| SYS-21 | For every claimed visual defect class, the team **shall** document the smallest demonstrated observable feature size, tested view locations, camera settings, image-processing conditions, and held-out detection performance. | Maintenance need to understand inspection limits; proposed verification requirement |

The proposed measurable ML targets distinguish inspection speed, component localization, and actual fault recognition. The five-minute target covers the full requested inspection cycle, rather than model inference alone. A 90% component recall target does not imply 90% detection of chipped blades or thermal abnormalities. The false-`PASS` objective applies to deliberately seeded conditions within the approved mandatory-check scope. All targets are preliminary and must be reviewed with Valinor, then evaluated on data collected from representative dock sessions before any operational performance claim is made.

The requirements above define a proposed acceptance baseline. The client must still identify the exact airframe, mandatory component list, acceptable failure response, meaningful defect sizes, flight-authorization boundary, and any stricter cycle-time or accuracy requirements.

**Baseline scope and stretch features:** The required software baseline consists of sensor acquisition and quality checks, RGB component localization, identification of required locations that cannot be evaluated, calibrated thermal-region assessment where coverage permits, conservative decision-level fusion, and a local versioned JSON result. Learned crack/chip classification using Anomalib or another anomaly-detection approach, additional learned fusion, active acoustic diagnosis, and directional-light defect enhancement are stretch features. These features shall not be prerequisites for demonstrating the core inspection. The team and Valinor will establish a documented go/no-go date for each stretch feature before the detailed-design schedule is finalized; no stretch feature will be treated as a committed capability without sufficient data, integration time, and safety approval.

### 1.2 Stakeholders and their needs

The proposed inspection system affects several groups with different operational, safety, and maintenance needs. These needs guide the requirements and the system's choice to report uncertainty explicitly.

| Stakeholder | Relevant needs and concerns | Related system requirements |
|---|---|---|
| Valinor engineering and dock-integration staff | Reliable integration with existing dock hardware and software; maintainable interfaces; no interference with safety-critical dock functions | SYS-05, SYS-08, SYS-10, SYS-13, SYS-16 |
| Remote pilots and dock operators | Timely, interpretable inspection outcomes; visible limitations and reasons for warnings; no false indication that the aircraft has been cleared for flight | SYS-01, SYS-06, SYS-07, SYS-11 |
| Maintenance technicians | Component-specific findings, preserved evidence, relevant reason codes, and a practical way to review possible faults without unnecessary repeated inspections | SYS-06, SYS-08, SYS-09, SYS-12 |
| Valinor's customers | Predictable system availability, traceable inspection records, and responsible handling of maintenance information | SYS-09, SYS-11, SYS-13 |
| People near flight operations | Conservative handling of missed or uncertain defects, and reasonable protection from unnecessary image collection | SYS-07, SYS-10, SYS-19 |
| Federal Aviation Administration (FAA) | An operating workflow that supports the remote pilot's legal inspection responsibilities rather than assigning flight authority to an AI model | SYS-01, SYS-07, SYS-10 |

The dock controller is an interfacing subsystem rather than a person or organization. It must supply an authorized inspection state and receive versioned, machine-readable results, while retaining operational control over dock functions. The proposed requirements remain subject to review with Valinor before they are treated as client-approved acceptance criteria.

### 1.3 Known constraints, measurements, and unresolved source differences

| Subject | Team-provided information | Design interpretation and next action |
|---|---|---|
| Dock enclosure | Approx. **30in W × 40in L × 12-16in H** [5][6][13][19][27][11][32] | The horizontal footprint is a working basis, not a measured installation drawing; usable internal height, doors, fixtures, and interference envelope must be measured. |
| Drone footprint | Approx. **20in × 20in**; height given as 6-8 inches in acoustic/camera draft [19][27][11][32] | Centered horizontal clearances would be 5 in across width and 10 inhes along length before fixtures; the camera draft reverses these labels in one equation, so a measured CAD layout must resolve the geometry. |
| Edge computer | Valinor identified an **NVIDIA Jetson Orin NX 16 GB** as the existing compute module. | The installed JetPack/L4T image, carrier board, CUDA, TensorRT, DeepStream, storage and available CPU/GPU/memory headroom must still be inventoried. The team shall not assume that upgrading ML libraries is safe for current dock software. [21] |
| Camera candidate | Four proposed USB 3.0 cameras based on a **16 MP Sony IMX519** sensor, subject to model/lens confirmation; provisional **4 pixels/mm** critical-region sampling goal. | Verify the exact vendor UVC module, supported output formats, field of view, lens, cost, and sampling at actual installed positions before design approval. |
| Thermal candidate | One radiometric Lepton 3.5, 160 × 120 array, approx. 57° horizontal field; PureThermal 3 USB board [5][6][13] | Temperature mode, raw-unit conversion, reference calibration and region coverage require bench verification. |
| Lighting | Two opposed **1m / 12V / 10W per m** strips, nominal 20W lighting load, independent switched channels [7][12] | A proposed 12V, 3A regulated lighting supply provides 36W nameplate capacity; verify actual current and temperatures. |
| Acoustic | One central USB microphone; proposed 2–3s buffers and low-RPM spin test [19][27] | Microphone can be prototyped with recorded/bench sounds; no autonomous in-dock motor spin is authorized by this document. |

**Computer compatibility decision gate:** Valinor identified the existing compute module as the **NVIDIA Jetson Orin NX 16 GB**. The original Jetson Nano's DeepStream 6.0.1 / JetPack 4.6.1 combination is not a suitable basis for selecting an Orin NX deployment environment. NVIDIA's current compatibility documentation must be checked against the actual installed JetPack/L4T, CUDA and TensorRT versions before pinning DeepStream and GStreamer dependencies. The team will inventory the dock's existing software and resource use first, then select a compatible tested inference stack without changing sponsor-controlled dock software without approval. [21]

**Valinor interface and data limitations:** Valinor reported that the Orin NX 16 GB is mounted on the backplate where the landing platform retracts. The existing software performs dock-state monitoring, including current-based detection associated with drone presence. Valinor will not provide its proprietary docking source code or dock design files and currently has no labeled PASS/WARN/FAIL drone-image or thermal dataset for this project. The inspection integration shall therefore use an approved black-box interface and data gathered by the team; exact dock power, mounting permissions, and software-service boundaries remain unresolved. Aircraft-specific NDA information shall not be included in public code or documentation. These facts are recorded as client-meeting information, not third-party published specifications.


## 2. Comparative analysis of possible solutions

Each comparison weighs inspection coverage, ease of integration, risk, development time, repeatability, and the team's limited semester resources. A selected concept is a starting point for testing, not proof that the performance requirement is met.

### 2.1 Visible-camera position and interface

| Option | Benefits | Limitations | Selection |
|---|---|---|---|
| **Four caddy-corner UVC cameras** | Multiple perspectives; shorter occluded regions; standard Linux V4L2/USB handling | Four mounts/cables, shared USB loading, cost, focus and overlap validation | **Selected prototype layout** [11][32] |
| Two opposite wall cameras | Fewer sensors and simpler cabling | Greater hidden areas; short working distance can require wide-angle optics | Not selected for initial full component survey |
| One overhead camera | Minimal interface complexity | Propeller undersides, sidewalls, motor housings may be blocked | Not sufficient alone |
| CSI/MIPI camera set | Dedicated camera interface can reduce USB dependence | Connector/driver integration and wiring constraints | Retained as fallback if UVC testing fails |

The source camera layout uses an approximately 25 in corner-to-center distance and an estimated 59° field-of-view concept [11][32]. Real usable field of view depends on tilt, lens distortion, focus, camera elevation, and the height of the propellers; it shall be checked with the real drone instead of accepted as a final optical calculation. [32][33]

The team is evaluating a 16 MP Sony IMX519-based USB 3.0 camera module and a preliminary sampling goal of **4 pixels/mm** in critical regions, subject to verification of the actual USB module, lens, output resolution and geometry. At 4 pixels/mm, a 0.5 mm feature occupies only 2 pixels, while a feature would need to be 0.75 mm wide to occupy 3 pixels:

\[
\text{sampled width at 0.5 mm}=4\ \text{pixels/mm}\times0.5\ \text{mm}=2\ \text{pixels}
\]

\[
\text{feature width for 3 pixels}=3/(4\ \text{pixels/mm})=0.75\ \text{mm}
\]

These are sampling calculations, not proof of crack, chip or scratch recognition. The effective detail level also depends on lens focus, perspective, glare, noise, JPEG compression, model input resizing and any high-resolution crop analysis. The RGB team shall measure sampling at near, center and far component locations. The ML team shall determine the smallest defect size that can actually be classified under representative conditions. The vendor's exact IMX519 USB module specifications must replace the unrelated IMX500 reference currently listed as [4] before final hardware selection.

**Camera selection note:** A 16 MP IMX519-based USB 3.0 module is a proposed research candidate, not a verified or purchased Valinor camera. The legacy IMX500 reference [4] describes a different sensor and must not be used as proof of this candidate's USB interface. Exact vendor SKU and lens are awaiting confirmation.


### 2.2 Lighting configuration

| Option | Strength | Weakness | Selection |
|---|---|---|---|
| **Opposed diffused white LED strips** | Low cost, repositionable, reduces hard shadows in many orientations | Diffuser/strip width may leave glare or dark regions | **Baseline inspection mode** [7][12] |
| Shallow-angle directional lighting | Can improve contrast around edges/chips/scratches | Additional switching and extra capture time; reflective surfaces behave differently | **Experimental second mode** |
| Ring/dome illumination | Potentially more even field | Additional space and hardware cost | Fallback if strips cannot meet uniformity requirements |
| Uncontrolled ambient illumination | No additional hardware | Exposure and defect contrast change with environment | Rejected |

The selected lighting is controlled through rated switching hardware. A Jetson logic output does not supply LED load current. The initial two-strip estimate is \(P = 2(10\,\mathrm{W}) = 20\,\mathrm{W}\) and \(I \approx 20/12 = 1.67\,\mathrm{A}\) total. With one independently switched 1 m strip per Adafruit 5648 driver, the estimated 0.83A/channel is below the board's 1.5 A continuous published rating; switching hardware still needs a thermal and connector-rating check. [7][12] [41][42]

### 2.3 Thermal sensing

| Option | Benefit | Drawback | Selection |
|---|---|---|---|
| **Lepton 3.5 + PureThermal 3 (USB)** | Radiometric-capable core; convenient USB integration; separate thermal pipeline | 160 × 120 pixels may not resolve small parts; surface readings need calibration | **Selected candidate** [5][6][13] |
| Lepton + low-level breakout/SPI | Fewer interface components | Increased firmware and capture-integration work | Not selected for first prototype |
| Several thermal cameras | Better coverage of multiple components | Significantly higher cost and calibration burden | Consider only after single-camera coverage mapping |
| RGB-only temperature inference | Lower cost | RGB intensities do not provide actual component temperature | Rejected |

The Lepton's approximate view at a 12 in stand-off is only part of the drone, so only mapped, visible thermal regions may receive a thermal assessment. The initial target regions are motor housings, the external battery casing area, and exposed charging connections, if visible. Battery-case surface temperature is not equivalent to internal cell temperature. Reflections, emissivity, time since flight, and charging state constrain comparisons. [5][6][13] [60]

### 2.4 Acoustic measurement and safe actuation

| Option | Benefit | Drawback | Selection |
|---|---|---|---|
| **USB Audio Class microphone via ALSA** | Linux-native device endpoint and simple removable hardware | Acoustic signal can be weak or confounded in a dock | **Selected research interface**, conditional operation [19][27] |
| I2S MEMS microphone | Avoids the USB hub | Device-tree/pinmux and physical wiring complexity | Not selected |
| No audio subsystem | No motor-actuation risk or added cost | May miss a fault detectable only during motion | **Safe baseline inspection** until testing is authorized |

The acoustic research proposes short, approximately 2–3s recordings during controlled low-RPM motor tests and feature extraction with Librosa/SciPy followed by a lightweight classifier [19][27]. This differs from the ML draft's explicit decision to **exclude automatic motor spin** [3][9]. The consolidated plan therefore treats audio as Phase B, not a normal inspection prerequisite. A separate, sponsor-approved safe procedure is required before any motor-related test; recorded or laboratory-generated clips can be used for initial software validation. [27][28]

### 2.5 AI processing, component analysis, and sensor fusion

| Design choice | Advantages | Challenges | Decision |
|---|---|---|---|
| **GStreamer/DeepStream + TensorRT + a benchmarked object detector** | Supports GPU-accelerated video handling, inference and component metadata on a compatible Jetson software stack | Requires confirmed Orin NX runtime versions, model conversion/parser validation, and resource testing alongside dock functions | **Provisional deployment architecture**; YOLOv4 remains the initial detector baseline, with newer models evaluated before final selection [3][21][22][23] |
| OpenCV-only pipeline | Quick camera and preprocessing prototype | More custom integration for multi-stage optimized inference | Development/testing support |
| YOLOv4-tiny | Lower compute and memory pressure | Potential loss of small-object recall | Fallback if full YOLOv4 misses timing/resource limits |
| Anomalib (PatchCore/PaDiM) on RGB crops | Can explore subtle deviation from healthy appearances | Dataset, deployment, and false alarm work; different runtime path | Stretch goal only [11][32] |
| **Late (decision-level) RGB/thermal fusion** | No requirement for pixel-identical RGB and thermal tensors; testable reason codes | Requires calibrated region mapping and decision rules | **Selected** [3][9] |
| Early/feature-level RGB–thermal fusion | Potential richer learned correlations | Strict alignment/training demands and greater integration risk | Not selected initially |

YOLOv4 is first used to locate components and generate named regions of interest. Detecting a component is not equivalent to validating that it has no defect. Surface damage classification will be attempted only for defect categories supported by labeled test examples and measured performance. A rules-based decision layer combines visible and thermal observations without claiming that a detection confidence is a probability of safe flight. [3][9]

### 2.6 Weighted software and compute trade studies

The AI architecture must balance detection quality with integration effort, available compute, software licensing, energy use, and the need to protect existing dock functions. The following weighted comparisons are preliminary engineering judgments, not measured test results. Final selections will be revisited using sponsor input and benchmark data from the installed Jetson Orin NX 16 GB.

#### Object detector alternatives

The initial concept selected YOLOv4 because the team had already researched the Darknet implementation and its TensorRT/DeepStream integration path. However, selecting YOLOv4 without evaluating newer detectors would not adequately justify the choice. YOLOX-S and Ultralytics YOLOv8 are included as more recent comparison candidates. License compliance must cover code, pretrained weights and dependencies separately; use of an open repository does not automatically permit unrestricted proprietary distribution. [3][22][23]

| Detector | License/IP compatibility (30%) | Current integration readiness (30%) | Potential small-part detection (25%) | Potential Jetson efficiency (15%) | Weighted score / 5 |
|---|---:|---:|---:|---:|---:|
| YOLOv4 / Darknet | 4 | 5 | 3 | 3 | 3.90 |
| YOLOX-S | 5 | 3 | 4 | 4 | 4.00 |
| Ultralytics YOLOv8 | 2 | 4 | 4 | 4 | 3.40 |

Ratings use a preliminary 1–5 scale, where 5 is more favorable. The weighted score is `0.30 × licensing + 0.30 × integration + 0.25 × potential detection + 0.15 × efficiency`. The values above represent planning assumptions and **do not establish that one model is more accurate or faster in the Valinor dock**. YOLOv4 remains the baseline for early development; the final detector choice will depend on common training/evaluation data, held-out dock sessions, inference timing, memory usage, export compatibility, small-component recall and license review. The Apache-2.0-licensed YOLOX implementation is a candidate for comparison, while Ultralytics software may require AGPL-3.0 obligations or an appropriate commercial license depending on intended use. [22][23]

#### Shared versus dedicated compute

| Decision criterion | Weight | Shared existing Orin NX 16 GB | Separate inspection compute module |
|---|---:|---|---|
| Avoidance of dock-control timing and reliability interference | 35% | Must be proven under simultaneous dock and inspection workloads | Greater isolation, but introduces a new dock connection |
| Incremental hardware, power and maintenance cost | 30% | Lower initial equipment cost if adequate headroom exists | Additional hardware, power and enclosure integration |
| Software and electrical integration effort | 20% | Shares existing OS and resource dependencies | Requires separate supply, data link and maintenance |
| ML throughput and future expansion headroom | 15% | Limited by currently available resources | Can be sized to the inspection workload |

A numerical compute winner is not assigned until the installed board's baseline utilization and the sponsor's allowed dock impact are known. The team's initial preference is to reuse the Orin NX if tests establish acceptable dock behavior; dedicated compute remains a fallback if resource contention creates unacceptable operational risk. Cost, power consumption and software integration are considered here as design-selection factors, rather than only as later broader-impact concerns.

The camera selection requires a separate weighted matrix led by the RGB subsystem owner, with coverage, spatial sampling, USB reliability, cost and installation effort mapped to the system requirements.

### 2.7 Existing automated visual inspection approaches

Published inspection work provides examples of how machine vision can identify damage or support maintenance review. However, reported results on other aircraft, cameras, and defect datasets cannot be assumed to apply to the Valinor dock.

| Existing approach | Relevant contribution | Limitation relative to this project |
|---|---|---|
| Harras et al., *Vision-based Propeller Damage Inspection Using Machine Learning* (2023) | Uses visual propeller data and a convolutional neural network to investigate automatic propeller-damage recognition; reports 85.8% accuracy in the paper's study | Does not demonstrate the proposed fixed multi-camera, RGB/thermal, dock-safe reporting system or its performance on the Valinor airframe [24] |
| Rice et al., *Automated Multi-Camera Inspection System for Aircraft* (AAAI 2026) | Demonstrates multi-camera, pan/tilt/zoom visual inspection and AI-assisted identification of aircraft exterior conditions with operator review | Uses a larger-airframe and motorized-camera inspection setting rather than a compact stationary drone dock [25] |
| Ishikawa, Iwata, and Satoh, *Event-Camera-Based Defect Detection of High-Speed Rotating Propellers* (2026) | Studies event-based sensing and classification of defects on rotating propellers | Requires a different sensing and motion scenario; the proposed baseline inspection does not energize propellers [26] |

These approaches support studying automated inspection but do not establish the accuracy, coverage or safety of the proposed device. The planned contribution is a compact, stationary in-dock system that combines mapped RGB component checks, limited validated thermal regions, explicit `UNVERIFIED` handling for incomplete evidence, and a versioned result communicated to the existing dock controller. Its effectiveness will depend on representative in-dock data and testing, not the performance numbers reported by unrelated research. [24][25][26]

## 3. High-level solution

The inspection manager receives a request from the dock controller, verifies a safe state, and assigns a unique run ID. The thermal camera captures a frame before visible lighting is energized. The controller then enables the general LED mode and the system records one camera at a time, with the capture mode, stable-light confirmation, and timestamps attached to accepted frames. An optional directional-light pass adds frames only if trials show a measurable benefit. Quality checks reject blurred, clipped, stale, obstructed, or incorrectly exposed images. [3][9]–[11][32]

Next, the software undistorts and maps the valid RGB images, runs the component detector, and checks whether mandatory parts are actually observable. The thermal adapter returns valid surface measurements and region identifiers without requiring a full pixel-to-pixel overlay. A decision manager combines component evidence with configuration-controlled severity rules. The result and supporting evidence are saved locally, transmitted to the dock, and displayed using the dock's existing maintenance/status interface. A failure to observe a required area is `UNVERIFIED`, not a successful inspection. [3][9][5][6][13]

### 3.1 Hardware block diagram

Each physical block below is a distinct hardware or dock subsystem. Solid arrows show required signals/power; the dashed microphone path is conditional.

```mermaid
flowchart LR
    DC["Dock controller / safety interlocks"]
    PS["Regulated dock power and protection"]
    LP["12V lighting supply"]
    SW["Two independently controlled LED switches"]
    LED["Diffused white LED fixtures"]
    RGB["Four corner UVC RGB cameras"]
    TH["Lepton 3.5 + PureThermal 3"]
    MIC["USB microphone - optional"]
    HUB["Externally powered USB hub"]
    J["Existing Jetson Orin NX 16 GB"]
    STORE["Local persistent storage"]
    UI["Dock status / maintenance UI"]
    PS -->|"Rated board input power"| J
    PS -->|"Approved DC feed"| LP
    LP -->|"12V DC, fused/rated"| SW
    DC -->|"Interlock-safe enable request"| J
    J -->|"3.3V logic or approved interface"| SW
    SW -->|"Switched LED power"| LED
    RGB -->|"UVC video over USB"| HUB
    TH -->|"Thermal UVC data over USB"| HUB
    MIC -.->|"USB Audio Class PCM"| HUB
    PS -->|"Rated hub DC power"| HUB
    HUB -->|"USB data"| J
    J -->|"Local JSON status + reason codes"| DC
    J -->|"Images / metrics / log files"| STORE
    DC -->|"Status and service prompt"| UI
```

**Block boundaries and wiring constraints:** The dock's existing controller has final authority over operational safety. The Jetson controls the lighting only through an approved driver and a controller-authorized inspection state; a safe de-energized state is required on crash, timeout, or aborted inspection. The hub's upstream connection, number of true root-bus controllers, and available bandwidth shall be measured; sequential acquisition reduces simultaneous traffic but does not guarantee zero USB contention. The thermal core/USB board may require an independent stabilized power path through the hub. Lighting power and Jetson GPIO shall never be directly tied at incompatible voltage levels. [7][12][19][27][11][32]

### 3.2 Processing decomposition and subsystem data flow

```mermaid
flowchart LR
    A["Inspection manager"] --> B["Sensor acquisition & quality"]
    B --> C["Visual condition checks"]
    C --> D["RGB preprocessing / calibration"]
    D --> E["YOLOv4 component detector"]
    B --> F["Thermal ROI / calibration adapter"]
    B -.-> G["Optional acoustic features"]
    E --> H["Decision / fusion rules"]
    F --> H
    G -.-> H
    H --> I["Status and JSON contract"]
    I --> J["Dock controller"]
    I --> K["Operator UI"]
```

An internal data record includes `inspection_id`, `capture_time_utc`, `sensor_id`, `lighting_mode`, image shape/units, `calibration_version`, `quality_state`, and the final component mapping. The thermal and visible pipelines may have different resolutions and capture times; their association is by **component ID and capture metadata**, not an assumed matching pixel grid.

### 3.3 Operational flowchart from the user's perspective

```mermaid
flowchart TD
    A[Drone docked] --> B{Safe to inspect?}
    B -- No --> C[BLOCKED: show reason]
    B -- Yes --> D[Start inspection ID]
    D --> E[Thermal capture; lights off]
    E --> F{Still safe?}
    F -- No --> X[Abort capture; lights off]
    F -- Yes --> G[RGB capture; lights on]
    G --> H{Still safe?}
    H -- No --> X
    H -- Yes --> I{Required images valid?}
    I -- No --> J{Safe recapture allowed?}
    J -- Yes --> G
    J -- No --> U[UNVERIFIED: missing evidence]
    I -- Yes --> K[Analyze and fuse results]
    K --> L{Confirmed critical fault?}
    L -- Yes --> M[FAIL: maintenance review]
    L -- No --> N{Warning or suspected fault?}
    N -- Yes --> O[WARN: review finding]
    N -- No --> P[PASS: tested checks only]
    X --> Y[BLOCKED or UNVERIFIED; DOCK_STATE_LOST]
    C --> Z[Record and show outcome]
    U --> Z
    Y --> Z
    M --> Z
    O --> Z
    P --> Z
    Z --> Q[Send JSON; dock retains control]
```

**User interface states:** `READY`, `BLOCKED`, `CAPTURING_THERMAL`, `CAPTURING_RGB`, `ANALYZING`, `PASS`, `WARN`, `FAIL`, and `UNVERIFIED`. The AI supplies a data contract; the team is not assuming that a separate touchscreen is purchased. The UI should identify which component was examined, which data were missing, and why review is requested.

**Loss of inspection authorization:** The inspection manager shall verify the dock's authorized state before and during sensor capture. If authorization is withdrawn, the software shall stop additional capture and inference, request the lighting hardware's fail-safe off state, record `DOCK_STATE_LOST`, and suppress a `PASS` for the interrupted inspection. The resulting `BLOCKED` or `UNVERIFIED` outcome shall be sent to the dock if communications remain available. The hardware-independent lighting shutoff mechanism is specified by the electrical/integration subsystem and cannot rely solely on a running user-space process.

### 3.4 External signal and protocol interface register

| Link (direction) | Physical / protocol type | Data sent and received | Design constraints / owner |
|---|---|---|---|
| Dock controller → Jetson | Existing dock-local IPC, exact transport **TBD** (Unix domain socket or authenticated localhost API proposed) | Request: `drone_id`, safe-docked flag/state, desired check set, request timestamp; response acknowledgment | Integration owner confirms existing controller, authorization, timeout and retry behavior. |
| Jetson → dock controller | Bidirectional local IPC response | `inspection_id`, overall and per-part status, `reason_codes`, quality flags, durations, evidence handles | One schema version, validation, idempotent request IDs; no autonomous flight clearance. |
| Four cameras → hub/Jetson | USB UVC, V4L2 `/dev/video*`; compressed MJPEG or uncompressed YUYV as module supports | RGB frames, device ID, dimensions, capture timestamp | Candidate actual SKU/lens and native still resolution **TBD**; sequential poll and verify stale buffers. [11][32] |
| Thermal module → hub/Jetson | PureThermal 3 USB UVC; raw radiometric format after verification | Original 16-bit thermal samples if exposed, timestamp, format/scale, sensor state | Must confirm raw units, shutter/FFC effects and calibrated conversion before temperature thresholds. [5][6][13] |
| Optional microphone → hub/Jetson | USB Audio Class, ALSA `/dev/snd/*` | PCM audio, sample rate, channels, event ID, timestamp | Default disabled for flight-ready in-dock operation; rate 16/44.1/48 kHz subject to device support. [19][27] |
| Jetson → LED switches | Logic-level GPIO to electrically compatible driver, ideally gated by dock interlock | Channel enable/state; optional brightness duty cycle after compatibility tests | LEDs powered by separate 12V rail; independent fail-safe off; no direct load through GPIO. [7][12] |
| Regulated supplies → peripherals | Protected DC power | Voltage/current at selected board, hub, switches and fixtures | Fuse, cable gauge, grounds, connectors and heat checked in final electrical design. |
| Jetson → storage | Filesystem + structured JSON/SQLite | Images/crops, summaries, configuration and model checksums, failures, timestamps | Permissions, remaining space, retention duration and write-error behavior **TBD**. |

**Preliminary dock communication contract:** Each inspection request shall include a unique `request_id`, a `drone_id`, a requested-check list, a timestamp, and a current `safe_to_inspect` authorization supplied by the dock controller. Each result shall identify `schema_version`, `request_id`, `inspection_id`, timestamp, overall and per-component state, reasons, required-modality validity, evidence references and model/configuration versions. The dock and inspection process shall define a heartbeat, acknowledgement behavior, duplicate-request handling, retry limits and bounded timeouts before integration tests. Their numerical values remain to be agreed with Valinor. Invalid, stale, unauthenticated, timed-out or duplicate instructions shall not initiate an unauthorized inspection; loss of authorization during capture shall prevent a `PASS` and be logged as `DOCK_STATE_LOST`.

### 3.5 Provisional result message

The following message shows the expected shape only. Values and results are fictitious and are **not measurements or established safety thresholds**.

```json
{
  "schema_version": "0.2-draft",
  "request_id": "request-00042",
  "inspection_id": "inspection-00042",
  "drone_id": "aircraft-01",
  "timestamp_utc": "2026-10-23T20:00:00Z",
  "overall_status": "WARN",
  "components": [
    {
      "component_id": "front_left_propeller",
      "status": "PASS",
      "modalities_checked": ["rgb"],
      "reasons": [],
      "rgb_detection_confidence": 0.94
    },
    {
      "component_id": "rear_right_motor",
      "status": "WARN",
      "modalities_checked": ["rgb", "thermal"],
      "reasons": ["THERMAL_PEER_DIFFERENCE"],
      "thermal_peer_delta_c": 9.2
    }
  ],
  "quality": {"rgb": "VALID", "thermal": "VALID", "audio": "NOT_REQUIRED", "dock_authorization": "VALID_AT_START_AND_CAPTURE"},
  "performance": {"total_inspection_ms": 7800},
  "versions": {"detector": "yolov4-test", "calibration": "cal-01", "rules": "rules-01"},
  "evidence": ["inspection-00042/front_left_rgb.jpg"]
}
```

**Decision precedence:** A confirmed critical condition returns overall `FAIL` even if some other data are missing; the missing-data problem must remain visible in the report. Otherwise, a mandatory unverified check yields `UNVERIFIED`; validated suspicious conditions yield `WARN`; only fully completed required checks without flags yield `PASS`. `WARN` and `UNVERIFIED` **shall not** be interpreted as automatic clearance. Final severity mapping and any actual flight-block behavior require client sign-off.

## 4. Atomic subsystem specifications

This section decomposes the concept into implementable subsystems. Each subsection includes function, input/output boundaries, a proposed contract, shall statements, and a practical verification method. Values designated **TBD** must be fixed before detailed design is approved.

### 4.1 Dock safety, power and mechanical integration

**Function:** Safely host the inspection cycle inside the dock, provide the approved power/interface boundaries, and preserve normal docking and aircraft operations.

| Inputs | Outputs | Interfaces / protocols | Proposed checks |
|---|---|---|---|
| Dock-safe state, installed-board identity, power sources, enclosure dimensions, mounted sensor dimensions | Inspection enable, regulated and protected DC rails, sensor/lighting mounting positions, controller status | Dock IPC; DC power; mechanical brackets; approved logic-level interlock | Confirm 30 × 40 in planning footprint against physical dock; measure vertical clearance; inspect cable routing, fuse sizing, temperature and load behavior |

- **INT-01:** The integration subsystem **shall** expose a positive `safe_to_inspect` condition indicating the drone is secured and propellers are stopped before enabling capture.
- **INT-02:** The integration subsystem **shall** prevent active illumination and conditional test hardware from obstructing landing, door travel, ventilation, charging contacts, or service access.
- **INT-03:** The power design **shall** provide separately rated Jetson, hub, and 12V lighting supplies or suitably sized converters without applying 12V to 3.3V GPIO.
- **INT-04:** All switched LED outputs **shall** default to off if the inspection process crashes, stops receiving authorized commands, or loses the dock enable state.
- **INT-05:** The team **shall** inventory the NVIDIA Jetson Orin NX 16 GB module, carrier board, installed JetPack/L4T, CUDA, TensorRT, DeepStream compatibility, and current dock software before modifying the inspection runtime.
- **INT-06:** With the dock's existing workload active, the team **shall** measure CPU utilization, GPU utilization, RAM usage, thermals, sensor power, capture and inference latency, and any missed or delayed dock-control deadlines; a sponsor-agreed impact limit shall govern acceptance of shared computing.

**Verification:** Dimensioned photographs/CAD clearance review, power-on measurements, disconnect/fault tests, dock abort test, and unchanged baseline docking function.

### 4.2 RGB camera hardware and acquisition

**Function:** Provide four component views using fixed corner camera mounts and repeatable high-resolution still capture. [11][32]

| Inputs | Outputs | Interfaces / protocols | Design values |
|---|---|---|---|
| Camera IDs, mode requests, LED-stable event, calibration configuration, USB power | Native RGB still frames, camera ID, original pixel dimensions, timestamps, quality metadata | UVC/V4L2 through powered USB hub to Jetson; RGB buffers to preprocessing | Four planned USB 3.0 16 MP IMX519-based cameras (exact vendor module/lens to be verified); provisional 4 px/mm sampling target at critical locations; no claimed minimum defect size before testing |

- **RGB-01:** The system **shall** capture one camera at a time for the initial multi-camera pass and record the camera and lighting mode for each accepted image.
- **RGB-02:** The camera mounts **shall** provide unobstructed views of a client-approved component coverage map; invisible components **shall** be marked `NOT_VISIBLE` rather than inferred present.
- **RGB-03:** The acquisition software **shall** query and save the verified device resolution, pixel format and stable device identifier; it shall not assume `/dev/video0` always means the same camera.
- **RGB-04:** The acquisition software **shall** reject stale buffered frames after mode or exposure changes and shall mark unplugged cameras and incomplete transfers invalid.
- **RGB-05:** The camera hardware **shall** maintain repeatable focus and stable mounting during repeated simulated docking cycles.
- **RGB-06:** Each chosen camera/lens **shall** undergo field-of-view, geometric calibration and known-feature image tests at representative center, corner, and edge locations.
- **RGB-07:** The team **shall** measure any effect of USB hub sharing and sequential capture on total cycle time and report the observed maximum, rather than assume bandwidth is never limiting.
- **RGB-08:** For the selected IMX519-based USB camera SKU, the team **shall** document supported native resolutions, frame/still formats and USB speed, then test whether compressed MJPEG, any sensor downsampling, and shared-hub traffic degrade small-feature evidence or increase capture time. Measured native and model-input resolutions **shall** be recorded separately.

**Functional sequence:** `Select stable device → set mode/exposure → request frame → flush stale buffer → capture → timestamp and label → quality gate → release endpoint → next camera.`

**Verification:** Annotated coverage map, image-quality target captures under each light mode, cable-removal test, repeated mounting measurements, end-to-end capture timing and a full-resolution pixel-density study.

**High-resolution evidence handling:** An object detector may use resized images for component localization, but a small crack or chip can be lost when the original high-resolution view is downsampled. Candidate fine-defect analysis will therefore evaluate cropped native-resolution regions in addition to full-frame detector input. The team will compare native and compressed image captures for loss of edge detail before making a minimum-defect-size claim.

### 4.3 Controlled illumination

**Function:** Supply repeatable visible illumination without powering LEDs through the Jetson's GPIO. Diffused white light is the baseline; a shallow-angle mode is retained only after evidence shows better defect contrast. [7][12]

| Inputs | Outputs | Interfaces / protocols | Preliminary electrical specification |
|---|---|---|---|
| Authorized capture phase, GPIO-compatible switch commands, 12V supply | Light mode `OFF`, `GENERAL`, optionally `DIRECTIONAL`; ready/stabilized indication | 3.3V control signal through compatible MOSFET drivers; separate switched 12V load | Two 1m cool-white strips, 10W each; 20W total, 1.67A total; 12V 3A proposed supply [7][12] |

- **LGT-01:** The illumination subsystem **shall** maintain lights off during baseline thermal-first capture and between inspections.
- **LGT-02:** Each 1 m strip **shall** use a switching channel whose continuous current rating exceeds the measured channel load under operating temperature.
- **LGT-03:** The lights **shall** enter the specified capture mode and stabilize before a frame is accepted; frames from prior modes **shall** be rejected.
- **LGT-04:** The mounts **shall** allow diffusion/angle adjustments without covering cameras, charging hardware, or the drone's motion envelope.
- **LGT-05:** The team **shall** measure coverage, reference-area brightness variability, glare, surface contrast, flicker/banding, current and fixture heating across repeated cycles.
- **LGT-06:** Directional lighting **shall** only be added to normal inspection if testing shows a measurable benefit that justifies its time and thermal effects.
- **LGT-07:** The control circuit **shall** turn off the lighting when power, software, or dock inspection authorization fails.

**Functional sequence:** `Read permitted inspection state → select thermal OFF mode → request GENERAL → wait for stable capture condition → acquire RGB frames → optionally set DIRECTIONAL and repeat selected views → switch OFF.`

**Verification:** Lux/relative-intensity map or fixed-exposure reference images, known-defect contrast experiment, camera banding check, supply-current and driver-temperature measurements, lights-fail-off test.

### 4.4 Thermal imaging and surface-temperature adapter

**Function:** Produce time-stamped thermal measurements from visible and calibrated target regions such as motor housings, exposed battery casing and charging hardware. [5][6][13]

| Inputs | Outputs | Interfaces / protocols | Working parameters |
|---|---|---|---|
| Lepton 3.5 radiometric stream, calibration data, component ROI map, capture command, flight/charge state if available | Temperature summary per mapped component, capture time, validity/reason codes, optional review colormap | Lepton core → PureThermal 3 → USB UVC → Jetson/OpenCV/NumPy; numeric data to fusion | **160 × 120**, approx. **57°** horizontal FOV, approx. **8.6 Hz** nominal [5][6][13] |

- **THM-01:** The adapter **shall** preserve raw thermal measurement units and metadata separately from any false-color visualization.
- **THM-02:** The adapter **shall not** report physical temperatures until the USB stream encoding, radiometric mode, scale and offset have been established and checked against a reference.
- **THM-03:** The thermal mount **shall** provide a documented component coverage map and identify excluded regions where a surface is hidden or insufficiently resolved.
- **THM-04:** Each result **shall** include ROI ID, sample count, capture timestamp and a validity flag; disconnected, stale, saturated, or uncalibrated data **shall** be `UNVERIFIED` for required checks.
- **THM-05:** Temperature comparisons **shall** distinguish post-flight, charging and ambient reference conditions; thresholds **shall** be derived from measured healthy ranges and controlled targets, not invented from example values.
- **THM-06:** The team **shall** test reflected heat and residual light-fixture heating; the lights-off-first sequence **shall** be revised if those tests indicate interference.
- **THM-07:** The adapter **shall** provide per-ROI median/maximum temperature or peer-difference measurements only when sampling and calibration quality permit meaningful values.

**Functional sequence:** `Verify USB device and temperature mode → confirm lights OFF → capture thermal frame → detect invalid/stale data → apply unit conversion and calibration → map named ROIs → return measurements/quality to decision layer.`

**Verification:** Controlled known-temperature target, repeated readings, blocked-lens test, unplugged camera test, ROI footprint demonstration and matched light-on/off trials. Surface temperature must not be described as a direct internal cell or bearing temperature.

### 4.5 Acoustic capture and research classifier (conditional)

**Function:** Capture an identified sound event for possible later mechanical-condition analysis. This subsystem is **not a required pass criterion** for the baseline dock inspection. [19][27][3][9]

| Inputs | Outputs | Interfaces / protocols | Working parameters |
|---|---|---|---|
| Approved test-event ID, USB microphone input, timestamp, reference audio | PCM buffer, MFCC/Mel-spectrogram features, optional experimental anomaly score, quality state | USB Audio Class → ALSA → Python audio capture → Librosa/SciPy → optional small classifier | One central microphone proposed; approx. 2–3 s clip and device-supported 16/44.1/48 kHz sampling to be tested |

- **AUD-01:** The baseline system **shall** function and complete required checks with the acoustic stage disabled.
- **AUD-02:** The microphone interface **shall** enumerate through ALSA and shall log its physical device identity, format, buffer length and sample rate.
- **AUD-03:** The subsystem **shall** reject clipped, missing, or event-unmatched recordings as invalid rather than claim a diagnostic condition.
- **AUD-04:** The software **shall not** energize or command the drone motors. Any low-RPM diagnostic needs a separate approved controller procedure and safety analysis before it can become a production step.
- **AUD-05:** Experimental features/classifications **shall** be developed first using controlled recorded/bench events and separately held-out test sessions.
- **AUD-06:** Any eventual acoustic warning **shall** include the test event and its limitations and **shall not** override confirmed RGB/thermal failure evidence.

**Verification:** USB microphone capture demonstration, noise/clipping assessment, controlled audio dataset and format test; optional motor-excitation phase awaits sponsor safety approval.

### 4.6 RGB calibration, component detection and condition checks

**Function:** Convert acquired camera views to consistent coordinates; detect known drone components; distinguish missing/occluded parts from visually abnormal parts. [3][9][11][32]

| Inputs | Outputs | Interfaces / protocols | Proposed implementation |
|---|---|---|---|
| Valid RGB frames, intrinsic/extrinsic calibration, lighting metadata, YOLO weights, component list | Corrected images, mapped bounding boxes, class/confidence, part ID, `visible` flag, visual rule flags | OpenCV transforms and image-quality tests; YOLOv4/Darknet deployed via compatible DeepStream/TensorRT; ROI data to fusion | Classes proposed: propeller, motor, arm, body, battery area, landing gear; fasteners conditional on visibility |

- **VIS-01:** The preprocessing module **shall** apply documented calibration and quality gates consistently during training evaluation and deployment.
- **VIS-02:** Bounding boxes **shall** map back to original-frame pixels and retain camera ID and calibration version.
- **VIS-03:** YOLOv4 **shall** be the baseline component locator, with YOLOv4-tiny considered only after a recorded benchmark motivates the change.
- **VIS-04:** Component names such as `front_left_propeller` **shall** be defined separately from general class names such as `propeller` so identical parts can be evaluated individually.
- **VIS-05:** A component that cannot be observed because of occlusion, blur, or field-of-view limitations **shall** be reported `NOT_INSPECTED`. A missing-part warning **shall** only be produced when the expected component location is adequately visible and a validated positional/absence rule supports the result.
- **VIS-06:** Blade chips, cracks, scratches, delamination, and loose fasteners **shall** remain candidate defect classes until the team has documented labeling criteria, representative training examples, held-out evaluations, and an observable feature-size limit for each claimed class. Component localization alone **shall not** be represented as proof of defect detection.
- **VIS-07:** The validation report **shall** include class-specific precision, recall and mAP for component localization; measured false negatives; available defect-class results; false-`PASS` outcomes from seeded-fault inspections; original-frame feature sampling; and inference and full-cycle timing from the actual Jetson Orin NX 16 GB configuration. Model confidence thresholds **shall** be versioned and selected using held-out validation data rather than treated as a probability of aircraft safety.

**Functional sequence:** `Verify frame → check exposure/blur → undistort → select ROI → run detector → map coordinates → compare required component locations → optional supported defect analysis → per-part evidence and quality.`

**Verification:** Calibrated target, labeled ground-truth dock sessions, intentionally missing mock components where safe, known harmless defect samples, and repeatable inference benchmarking.

### 4.7 Multimodal fusion and diagnostic decision manager

**Function:** Map RGB and thermal information to the same named component and combine validated findings into a conservative, explainable inspection result.

| Inputs | Outputs | Interfaces / protocols | Main rule hierarchy |
|---|---|---|---|
| Visual component statuses, thermal ROI values/quality, optional approved audio flags, mandatory-check list, thresholds/config versions | Per-component and overall status, reasons, sensor coverage, supporting evidence references | Typed internal records → JSON report builder | Critical confirmed `FAIL` → `FAIL`; otherwise missing mandatory → `UNVERIFIED`; otherwise suspicious → `WARN`; else `PASS` |

- **FUS-01:** The fusion layer **shall** associate sensor results using a named component ID and documented geometric mapping; it shall not assume RGB and thermal pixels correspond one-to-one.
- **FUS-02:** Each mandatory component **shall** have a documented applicable-modality checklist; an inapplicable modality must not be treated as missing mandatory evidence.
- **FUS-03:** An unavailable required modality **shall** prevent a PASS for the affected component even when another modality reports normal data.
- **FUS-04:** A confirmed critical defect **shall** retain its FAIL status even if another sensor reports normal readings or becomes unavailable.
- **FUS-05:** Thermal alert limits, ML score cutoffs, and critical-component definitions **shall** be versioned and based on approved test evidence before operational use.
- **FUS-06:** Every non-PASS state **shall** include a reason, affected component, sensor quality and an action label suitable for maintenance review.

**Decision flowchart:**

```mermaid
flowchart TD
    A["Gather evidence for one required component"] --> B{"Confirmed critical condition?"}
    B -- Yes --> C["FAIL; preserve other quality flags"]
    B -- No --> D{"Mandatory checks complete and valid?"}
    D -- No --> E["UNVERIFIED; give missing-data reason"]
    D -- Yes --> F{"Validated warning rule triggered?"}
    F -- Yes --> G["WARN; identify suspected condition"]
    F -- No --> H["PASS for tested conditions only"]
    C --> I["Combine component results into overall status"]
    E --> I
    G --> I
    H --> I
```

**Verification:** Synthetic test matrices covering all PASS/WARN/FAIL/UNVERIFIED combinations, sensor faults, missing parts, contradictory signals and configuration-version changes.

### 4.8 Data records, model management, and dock interface

**Function:** Make the inspection result traceable, reviewable and compatible with current dock software. [3][9]

| Inputs | Outputs | Interfaces / protocols | Storage / UI behavior |
|---|---|---|---|
| Per-component findings, selected images, calibration and model versions, timestamps, performance measurements | JSON result, dock UI status and review prompts, local evidence and run log | Versioned local JSON over agreed dock IPC; local filesystem with SQLite/JSONL manifest | Retention/space budget and exact IPC method **TBD**; only necessary images are retained |

- **DAT-01:** The report **shall** include `schema_version`, `inspection_id`, `drone_id`, overall/per-component status, reasons, quality summary, and run timestamp.
- **DAT-02:** The report **shall** include detector and calibration versions and a way to identify the associated rules/threshold configuration.
- **DAT-03:** The dock interface **shall** validate malformed requests and support bounded timeout/error responses.
- **DAT-04:** The storage module **shall** record failed inspections and representative evidence while applying a documented capacity limit and deletion/retention procedure.
- **DAT-05:** The system **shall** mark failed storage writes and lost dock acknowledgments explicitly; it shall not treat a successful model inference as a successfully communicated inspection.
- **DAT-06:** Model replacement and calibration updates **shall** preserve the external message schema unless a versioned change is reviewed by integration owners.
- **DAT-07:** The user-visible status **shall** show a component name, inspection time, issue reason and an appropriate rescan/maintenance-review prompt for non-PASS outcomes.
- **DAT-08:** Each request and result **shall** include a stable request identifier and schema version, and the interface **shall** reject malformed, unauthorized, stale, and conflicting duplicate requests without starting a new inspection.
- **DAT-09:** The dock integration **shall** define and test heartbeat monitoring, bounded request/response timeouts, lost acknowledgement handling, and inspection authorization withdrawal before operational use. Approved numerical timeout limits remain pending sponsor interface review.
- **DAT-10:** If safe-state authorization is withdrawn during an inspection, the software **shall** stop acquisition, request lighting shutdown, record `DOCK_STATE_LOST`, preserve the interruption reason, and **shall not** return a completed `PASS` result.
- **DAT-11:** The evidence-storage plan **shall** estimate required capacity from the measured average size and number of retained RGB images, thermal records, crops, logs and inspection frequency, with a documented storage margin and sponsor-approved retention period.
- **DAT-12:** If storage is unavailable or full, the pipeline **shall** retain an explicit storage-error state and **shall not** silently claim that a complete traceable inspection was saved or delivered.

**Storage sizing method:** Planning capacity will be estimated as `daily inspections × average saved evidence bytes per inspection × retention days`, plus space for indexes, logs, model versions and a safety margin. The image count, compression format, daily usage, retention period and available dock storage are not yet confirmed. The team will measure actual output sizes after a camera/thermal prototype is available, propose an access policy and deletion schedule, and seek Valinor approval before setting a mandatory retention value.

**Verification:** Schema test against representative messages, malformed/duplicate-request tests, full-disk test, report review with dock stakeholder and model-version rollback demonstration.

### 4.9 Dataset, calibration, and verification infrastructure

**Function:** Provide controlled evidence that the chosen design works on the team's actual dock and drone rather than relying on benchmark claims from unrelated data.

| Inputs | Outputs | Interfaces / protocols | Proposed tools |
|---|---|---|---|
| Real dock images, safe simulated defects, calibration targets, measured temperature references, software builds | Annotated session dataset, calibration maps, test metrics, regression cases, release manifest | CVAT bounding-box export, OpenCV calibration, local training/deployment machine, test scripts | YOLO-compatible labels; held-out **capture sessions**, not near-duplicate video frames |

- **TST-01:** Training, validation and final test data **shall** be split by recording session or other meaningful condition so nearly identical adjacent frames do not leak into separate sets.
- **TST-02:** Each labeled required component **shall** have an unambiguous class, named position and annotation policy.
- **TST-03:** The tests **shall** include normal samples and safe, non-destructive simulated defects under varied camera angles, lighting, landing offsets and environmental conditions.
- **TST-04:** The test report **shall** separately report detection recall, visual-defect capability, thermal measurement repeatability, overall inspection time and the false-PASS rate on deliberately seeded faults.
- **TST-05:** Calibration and preprocessing parameters **shall** be versioned and retained with the associated evaluation result.
- **TST-06:** External datasets such as MVTec AD/LOCO AD **shall** be used only after checking their allowed use; they **shall not** replace dock-specific performance evaluation. [3][9]
- **TST-07:** The team **shall** report the number of independent held-out seeded-fault trials, the observed false-`PASS` count and a one-sided 95% binomial confidence bound, with results separated by fault category when sample counts permit.
- **TST-08:** The team **shall** track positive sample counts for each required component class and shall separate training, validation and final testing by capture session or comparable independent conditions, not adjacent images of the same event.

**Preliminary sample-size planning:** For a proposed maximum false-`PASS` probability of 5%, **59 independent seeded-fault inspections with zero false `PASS` outcomes** would give an exact one-sided 95% binomial upper bound below 5% (`1 - 0.05^(1/59) ≈ 4.95%`). This is an illustrative minimum for an aggregate fault test, not a substitute for sufficient examples of each fault type. For a 90% recall estimate with an approximate ±5 percentage-point margin at 95% confidence, a simple normal approximation requires approximately **139 independent positive examples per class** (`1.96² × 0.90 × 0.10 / 0.05² ≈ 138.3`). Actual test sizes may increase because of dataset correlations, class differences and observed performance. These numbers guide planning and are not results already achieved by the team.

**Verification:** Review dataset manifests, re-run the same evaluation on a held-out capture day, and reproduce the software/image-calibration environment from saved versions.



## 5. Ethical, professional, and standards considerations

### 5.1 Public safety, false negatives, and accountability

The inspection system may influence a maintenance decision, so the most serious project-specific risk is a false `PASS` on a damaged drone. The system therefore distinguishes *healthy according to tested checks* from *not inspected*, keeps missing inputs visible, rejects unsafe start conditions, and avoids direct authority over launch decisions. The team shall log reproducible evidence and report false-negative cases rather than stating that the model detects all cracks or damage. The NIST AI Risk Management Framework and IEEE 7000-2021 provide processes for considering safety, uncertainty and ethical risks while developing AI-assisted systems. [1][14]

Under **14 CFR § 107.15**, the remote pilot in command is responsible for checking the small unmanned aircraft system before each flight to determine whether it is in a condition for safe operation. The proposed in-dock inspection is intended to support that responsibility by identifying selected observable components, documenting possible abnormalities, and reporting incomplete checks. It does not certify the aircraft as airworthy, replace a required preflight assessment, or independently authorize launch. A `PASS` means only that the implemented and validated mandatory checks were completed without a flagged issue; `WARN`, `FAIL`, and `UNVERIFIED` remain visible to authorized operators and the dock controller. [20]

The ML verification plan will treat a false `PASS` on a seeded fault as a primary safety metric and will report both the observed rate and uncertainty due to sample size. Component recall, defect recall, invalid-image frequency, thermal-data limitations and incomplete inspections will be reported separately. An overall accuracy value will not be used to hide poor performance on a critical component or rare fault category. The preliminary 5% false-`PASS` objective is a proposed test target requiring sponsor review, not evidence that the completed system is safe for autonomous flight decisions.

### 5.2 Electrical, thermal, and physical safety

A metal enclosure containing USB-powered devices and 12V / 20W nominal lighting requires cable restraint, appropriate fusing, supply sizing, safe separation of loads from logic GPIO, ventilation and temperature checks. The lighting and optional audio equipment must remain outside the drone's landing envelope. Any motor-powered acoustic test raises an additional propeller-strike risk; this design disables it until a separate approved procedure exists. IEC 62368-1 is consulted as an electrical/equipment safety reference, not presented as a verified compliance claim. [7][12][19][27][11][32][15]

### 5.3 Measurement quality and misleading results

For RGB inspection, ISO 12233:2024 informs camera resolution/spatial-frequency testing, while ISO 15739:2023 helps the team frame camera noise/dynamic-range evaluation. For thermal inspection, ISO 18434-1/-2 and ASTM E1933/E1934 inform thermal condition-monitoring practice, interpretation and surface-emissivity considerations. These references shape the requirement to calibrate, document test conditions, and retain uncertainty, rather than authorize a universal temperature alarm. [5][6][13][11][32][16][17]

### 5.4 Cybersecurity, privacy, licensing, and responsible development

The dock interface shall validate locally received messages, restrict unnecessary network exposure and handle stale or repeated requests. IEC 62443-4-1:2018 supplies useful secure-development concepts. Stored images might include surroundings beyond the aircraft; the capture framing and retention policy shall minimize unnecessary collection, and access shall be limited to the development and maintenance purpose. ISO/IEC/IEEE 29148:2018 and IEEE 1012-2024 are relevant guides for testable requirements and verification planning. [2][18][19]

The AlexeyAB/Darknet code license supports the team's interest in modifiable source code, but it does not confer a license over all model weights, external datasets, other libraries or redistribution terms. A dependency and dataset license register will be created before client handoff. [3][9]

#### Applicable software standard and planned verification

| Standard | Influence on the proposed subsystem | Planned evidence or test |
|---|---|---|
| IEC 62443-4-1:2018, secure product development lifecycle requirements | Guides secure development of the dock-facing inspection software, including request checking, fault handling, change tracking and vulnerability remediation | Test malformed, stale, unauthorized and duplicate messages; record expected responses and software revisions; review versioned issue/remediation records [2] |

**Engineering process references:** NIST's AI Risk Management Framework and IEEE 7000-2021 inform risk and ethical review for false-`PASS` results, operator overreliance and uncertainty. ISO/IEC/IEEE 29148:2018 informs clear, traceable, verifiable requirements; IEEE 1012-2024 informs software verification and validation planning. These are treated as design/development process references, not counted as distinct physical-subsystem standards or as evidence of compliance certification. [1][14][18]

The RGB camera and thermal owners will separately map their relevant measurement standards to design choices and practical tests (for example, camera resolution/contrast measurements and thermal reference-target checks). The project does not claim third-party standards certification.

### 5.5 Effects on the client, environment and cost

A modular, locally processed design avoids requiring continual cloud connectivity and can reduce routine manual screening time, but it adds electronics, energy demand, replaceable camera hardware and maintenance work. Reusing the dock's actual Jetson prevents an unnecessary second compute purchase, provided performance measurements confirm capacity. False alarms could increase maintenance effort, while false negatives could create much more severe safety implications; evaluation must report both. The lighting is on only during image acquisition to reduce wasted energy and unwanted heat. [3][5][7][9][11][19]

To reduce unnecessary maintenance work, a flagged component will be accompanied by its location, modality, reason code, image or thermal evidence when available, and an explicit `WARN`, `FAIL`, or `UNVERIFIED` explanation. A bounded recapture may resolve a quality failure without creating repeated unexplained alarms; genuine suspected faults will remain available for human review. To reduce privacy impact on people near the dock, cameras should be aimed primarily at the aircraft and interior inspection area, external scene regions should be cropped or masked when practical, and only necessary diagnostic evidence should be retained with role-restricted access. The storage duration and access permissions will be agreed with Valinor rather than invented at the conceptual stage. These mitigations make maintainability, privacy and operational cost active design criteria alongside detector accuracy and hardware performance.

## 6. Resources

### 6.1 Budget proposal and subsystem justification

The budget below combines the team's **published quoted prices** for the thermal core, USB board and LED components with **clearly marked planning allowances** for items that were not quoted in the uploaded research. All values are in **USD**, before tax/shipping. Existing dock equipment is treated as sunk cost, not a new purchase. Camera part numbers, microphone selection and stock must be finalized before ordering. [7][12][5][6][13][11][32]–[7][12]

| Subsystem / item | Qty | Unit cost or allowance | Proposed total | Basis and purpose |
|---|---:|---:|---:|---|
| RGB: high-resolution UVC cameras, exact SKU TBD | 4 | $126 | $504 | Planning allowance, not a vendor quote; four exterior views |
| RGB: camera brackets/adjustable mounts | 4 sets | $10–$20 | $40–$80 | Allowance for corner alignment and retention |
| Thermal: FLIR Lepton 3.5 | 1 | $164 | $164 | GroupGets/FLIR listed candidate price [5] |
| Thermal: PureThermal 3 board | 1 | $119.99 ≈ $120 | $120 | GroupGets listed candidate [6] |
| Lighting: Adafruit 4612 white strip, 1 m | 2 | $11.95 | $23.90 | Two opposed diffused sources [7] |
| Lighting: Adafruit 5648 switch board | 2 | $3.95 | $7.90 | Independent LED switching [12] |
| Lighting: regulated 12V / 3A supply | 1 | $25–$45 | $25–$45 | protects required 20 W load |
| Lighting: diffusers and optical hardware | 1 set | $25–$50 | $25–$50 | Allowance for glare/contrast trials |
| Integration: externally powered USB hub | 1 | $50–$80 | $50–$80 | shared with RGB, thermal and audio |
| Integration: USB leads, wiring and protection | 1 lot | $30–$60 | $30–$60 | power and data connections |
| Lighting: adjustable fixture brackets | 1 lot | $30–$60 | $30–$60 | opposed positions/angles |
| Jetson and dock controller | 1 | $175-$250 incremental | $175-$250 | Confirm installed board and resource availability |
| **Mandatory prototype subtotal** | | | $1,350.80–$1,600.80 | Excludes optional modules and existing hardware |
| Optional acoustic: USB microphone | 1 | $20–$70 | $20–$70 | sponsor approval for active diagnostic TBD |
| Optional evidence storage / backup | 1 | $25–$60 | $25–$60 | only if existing storage is insufficient |
| **Subtotal including both optional allowances** | | | $$1,395.80–$1,730.80 | No double count of the common USB hub |
| OpenCV, GStreamer, compatible DeepStream, Darknet, CVAT, Python tooling | — | $0 planned software purchase | $0 | Validate download, dependency and usage licenses |

**Planning reserve:** A 15% allowance on the mandatory subtotal is*$119.37–$193.62, giving a mandatory planning request of about $915–$1,485 excluding shipping and tax. With *both* optional items included before the same 15% reserve, the total becomes approximately $967–$1,634. These are range estimates, not committed purchasing totals.

**Cost reduction order:** Use existing development computers, borrow test targets/calibration gear, prototype lighting fixtures before buying additional sources, and retain a single thermal camera unless actual coverage tests justify another. A second lighting mode, a second thermal view, or replacement edge compute are **not in the mandatory budget**.

### 6.2 Division of labor and skills-to-subsystem allocation

The supplied research demonstrates that the team has developed work in RGB/optics and UVC, lighting/electrical design, thermal measurement, acoustic/USB audio, and ML/Jetson deployment. These documents demonstrate research ownership but do not establish that each proposed owner has already completed hardware, calibration or software integration skills tests. Responsibility assignments below are proposed roles, pending confirmation of team membership and the sponsor's expectations. Only the name in the original ML draft is preserved; names of other researchers have not been invented. [3][5][7][9][11][19]

| Subsystem or deliverable | Proposed lead / assignment | Skills needed and early gap check | Main responsibility |
|---|---|---|---|
| ML component detector, datasets, runtime, fusion and reporting schema | **Lucas Taylor** | Python/C++, OpenCV, DeepStream/TensorRT, evaluation metrics and JSON contracts; coordinate dock IPC with assigned integration owner | Benchmark component detector and model resource use; define fusion/reporting behavior and ML evaluation plan |
| Four-camera optics, mounts and USB acquisition | RGB camera research contributor — **Aaron Thongmanivong** | Lens/FOV math, UVC/V4L2, calibration, mechanical mounting | Select verified UVC SKU; coverage and spatial-resolution test |
| LED layout and supply/switching | Controlled lighting research contributor — **Samuel Oke** | 12V sizing, MOSFET drivers, optical test rig, safe wiring | Build repeatable light fixture, current and contrast trials |
| Lepton/PureThermal calibration and ROIs | Thermal research contributor — **Michael Lamberth** | Radiometry, USB imaging, emissivity/temperature validation | Thermal capture, reference verification, component mapping |
| Acoustic proof of concept | Acoustic research contributor — **Aaron Thongmanivong** | ALSA, DSP, recording, safe test planning | Microphone capture and dataset; do not request in-dock motor actuation |
| Dock electrical/mechanical integration | To be reassigned among remaining team members | Jetson software inventory, hub power, team-created CAD/clearance plans, interlocks and dock IPC; coordinate with Valinor | Check enclosure, supplies, cable routing and existing dock status interface |
| System verification, Gantt and report review | **Whole team** | Test matrices, traceability, budgeting, Git collaboration | Collect evidence, verify cross-subsystem acceptance and own review signoffs |

**Team allocation rule:** The integration role is unassigned following Benjamin's departure; this change does not erase his historical October contribution statement. Each subsystem lead is responsible for its own detailed-design schematic/code, interfaces, test evidence and section in the final report. System-level testing and integration are shared; the test coordinator checks completeness but does not certify another member's individual contribution. 

### 6.3 Timeline and revised Gantt chart

The schedule below is a proposed Fall 2026 detailed-design work plan beginning after this October 23 draft. It does not assert a client-approved milestone date. The highest-risk unknowns—the actual Jetson board/software, four-camera view geometry, Lepton radiometric mode, and safe docking interface—are tested first because they constrain all dependent work. Prototype parts may be assessed on a bench before mounting them into the dock.

<p align="center">
  <img src="Research%20of%20Subsystems/GanttChart.png" alt="Project Gantt Chart" width="850">
</p>


<p align="center">
  <em>Project development timeline and Gantt chart.</em>
</p>


### 6.4 Risk register and remaining client decisions

| Risk / unanswered question | Impact | Early mitigation / decision owner |
|---|---|---|
| Existing Orin NX software compatibility and compute headroom not yet verified | An incompatible runtime or GPU/memory contention could interrupt normal dock operation | Inventory installed JetPack/L4T and current dock resource use; benchmark concurrent dock + inspection loads; retain separate compute as a fallback — ML and integration owners |
| Actual camera SKU and 0.5 mm detectability unknown | Missing required defect visibility | Mount/test one candidate at proposed positions with physical targets; RGB owner |
| Single thermal camera sees only selected areas | Incomplete heat inspection | Annotate real visible ROIs, quantify coverage, revise mandatory thermal list; thermal/client |
| Thermal raw units/emissivity/charging condition unknown | False alarms or incorrect interpretation | Reference test and condition-specific baseline; thermal owner |
| USB hub bus/power limitations | Dropouts or false missing-frame results | Test powered hub and bus topology under realistic loads; RGB/integration |
| Motor-based acoustic test not authorized | Safety and schedule risk | Default audio disabled; develop off-dock recorded/bench proof; acoustic/client |
| Component list and flight consequences undefined | Cannot finalize required checks/status severity | Client confirms mandatory parts, manual-review rule and dock response; whole team |
| Image data and pretrained weight licenses | Handoff/use restrictions | Maintain dependency/data register and review licenses; ML/integration |


## 7. References

### 7.1 Consolidated external technical references from the team research

The numbered references below retain the original conceptual design sources [1]–[19], include the selected review-guide sources [20]–[26], and consolidate the original external sources cited in the acoustic, camera, controlled-lighting, ML and thermal condensed research. A URL is listed only once where possible. Earlier Jetson Nano and IMX500 sources are retained only as historical background; they do not describe the installed Orin NX 16 GB or a confirmed IMX519 USB SKU. Internal team-draft citation placeholders were removed.

[1] NIST, [*AI Risk Management Framework Playbook*](https://www.nist.gov/itl/ai-risk-management-framework/nist-ai-rmf-playbook).

[2] IEC, [*IEC 62443-4-1:2018, Secure Product Development Lifecycle Requirements*](https://webstore.iec.ch/en/publication/33615).

[3] A. Bochkovskiy, C.-Y. Wang, and H.-Y. M. Liao, [*YOLOv4: Optimal Speed and Accuracy of Object Detection*](https://arxiv.org/abs/2004.10934), 2020; AlexeyAB, [Darknet source/license](https://github.com/AlexeyAB/darknet/blob/master/LICENSE).

[4] Sony, [*IMX500/IMX501 Intelligent Vision Sensor specifications*](https://www.sony.com/en/SonyInfo/News/Press/202005/20-037E/), 2020 (lists 12.3 MP output and MIPI D-PHY/SPI interface). Historical alternative only; not proof of the proposed IMX519 USB camera.

[5] Teledyne FLIR, [*Lepton camera specifications*](https://oem.flir.com/products/lepton/); GroupGets, [*FLIR Lepton 3.5*](https://groupgets.com/products/flir-lepton-3-5) (radiometric 160 × 120, 57°, listed price $164 at time checked).

[6] GroupGets, [*PureThermal 3*](https://groupgets.com/products/purethermal-3) (USB interface board, listed price $119.99 at time checked).

[7] Adafruit, [*Ultra Flexible White LED Strip, product 4612*](https://www.adafruit.com/product/4612), and [linked datasheet](https://cdn-shop.adafruit.com/product-files/4612/C15482-001__6500K_.pdf).

[8] NVIDIA, [*DeepStream legacy Jetson/JetPack compatibility table*](https://docs.nvidia.com/metropolis/deepstream-nvaie31/dev-guide/text/DS_Quickstart.html) (original Nano / DeepStream 6.0.1 / JetPack 4.6.1).

[9] NVIDIA, [*DeepStream platform and operating-system compatibility*](https://docs.nvidia.com/metropolis/deepstream/8.0/text/DS_Installation.html) (Orin Nano support in earlier Orin-oriented releases including DeepStream 7.1 / JetPack 6.1).

[10] NVIDIA, [*Jetson Orin Nano Super Developer Kit specifications*](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/nano-super-developer-kit/), and [*JetPack 6.2 SDK*](https://developer.nvidia.com/embedded/jetpack-sdk-62).

[11] ISO, [*ISO 12233:2024, Digital cameras — Resolution and spatial frequency responses*](https://www.iso.org/standard/88626.html); OpenCV, [camera calibration](https://docs.opencv.org/4.x/d9/d0c/group__calib3d.html).

[12] Adafruit, [*MOSFET Driver, product 5648*](https://www.adafruit.com/product/5648) (1.5 A continuous load, $3.95 per board at time checked).

[13] Teledyne FLIR, [*Radiometry with Lepton*](https://oem.flir.com/learn/thermal-integration-made-easy/radiometry-with-lepton/).

[14] IEEE Standards Association, [*IEEE Std 7000-2021, Model Process for Addressing Ethical Concerns during System Design*](https://standards.ieee.org/ieee/IEEE-7000/6781/).

[15] IEC, [*IEC 62368-1, Audio/video, information and communication technology equipment — Safety requirements*](https://webstore.iec.ch/en/search?query=IEC%2062368-1).

[16] ISO, [*ISO 15739:2023*](https://www.iso.org/search.html?q=ISO%2015739%3A2023); [*ISO 18434-1:2008*](https://www.iso.org/standard/41648.html) and [*ISO 18434-2:2019*](https://www.iso.org/standard/67617.html).

[17] ASTM, [*E1933-14(2022)*](https://store.astm.org/e1933-14r22.html) and [*E1934-99a(2024)*](https://store.astm.org/e1934-99ar24.html).

[18] ISO, [*ISO/IEC/IEEE 29148:2018, Requirements engineering*](https://www.iso.org/standard/72089.html); IEEE, [*1012-2024*](https://standards.ieee.org/ieee/1012/7324/).

[19] USB Implementers Forum, [*USB Audio Class 2.0 specification*](https://www.usb.org/document-library/usb-audio-devices-rev-20-and-adopters-agreement); CVAT, [YOLO annotation format](https://docs.cvat.ai/docs/dataset_management/formats/format-yolo/).

[20] Electronic Code of Federal Regulations, [*14 CFR § 107.15 — Condition for safe operation*](https://www.ecfr.gov/current/title-14/chapter-I/subchapter-F/part-107/subpart-B/section-107.15), accessed November 2026.

[21] NVIDIA, [*Jetson Orin product family*](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/), accessed November 2026; for installation compatibility see [9]. Installed release not confirmed.

[22] Megvii-BaseDetection, [*YOLOX implementation and Apache-2.0 license*](https://github.com/Megvii-BaseDetection/YOLOX); Ultralytics, [*Licensing*](https://www.ultralytics.com/license), accessed November 2026.

[23] Ultralytics, [*Ultralytics YOLO implementation*](https://github.com/ultralytics/ultralytics), accessed November 2026. For Darknet implementation and license see [3].

[24] M. S. Harras, S. Saleh, B. Battseren, and W. Hardt, [*Vision-based Propeller Damage Inspection Using Machine Learning*](https://www.bibliothek.tu-chemnitz.de/ojs/index.php/cs/article/view/604), *Embedded Selforganising Systems*, vol. 10, no. 7, 2023, doi:10.14464/ess.v10i7.604.

[25] M. D. Rice et al., [*Automated Multi-Camera Inspection System for Aircraft*](https://ojs.aaai.org/index.php/AAAI/article/download/42378/46339), *Proceedings of the AAAI Conference on Artificial Intelligence*, 2026, doi:10.1609/aaai.v40i48.42378.

[26] Y. Ishikawa, K. Iwata, and Y. Satoh, [*Event-Camera-Based Defect Detection of High-Speed Rotating Propellers*](https://www.mdpi.com/1424-8220/26/15/4721), *Sensors*, vol. 26, no. 15, article 4721, 2026, doi:10.3390/s26154721.

[27] TDK InvenSense, "INMP441: Omnidirectional Microphone with I2S Digital Output Data Sheet," TDK Corporation, Document DS-INMP441, Rev. 1.1, 2014. [Original source](https://invensense.tdk.com/download-pdf/inmp441-datasheet/). (From Acoustic condensed research.)

[28] NVIDIA Corporation, "NVIDIA Jetson Linux Developer Guide: Configuring the 40-Pin Expansion Header and Device Tree Overlays," NVIDIA Docs, 2024. [Original source](https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/HR/ConfiguringThe40PinExpansionHeader.html). (From Acoustic condensed research.)

[29] Edge AI and Vision Alliance, "Unlock Edge AI Potential: Why USB Cameras and Sensors Are the Future of Vision Solutions," Edge AI and Vision Alliance, Nov. 2023. [Original source](https://www.edge-ai-vision.com/2023/11/unlock-edge-ai-potential-why-usb-cameras-and-sensors-are-the-future-of-vision-solutions/). (From Acoustic condensed research.)

[30] J. Scanlon and B. Sadler, "Acoustic Detection and Classification of Micro Air Vehicles (MAVs)," in Proc. IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP), 2014, pp. 2784–2788. [Original source](https://doi.org/10.1109/ICASSP.2014.6854107). (From Acoustic condensed research.)

[31] B. McFee et al., "librosa: Audio and Music Signal Analysis in Python," in Proc. of the 14th Python in Science Conference, 2015, pp. 18–24. [Original source](https://doi.org/10.25080/Majora-7b98e3ed-003). (From Acoustic condensed research.)

[32] Edge AI and Vision Alliance, "What is a UVC Camera, and What are the Different Types of UVC Cameras?," Edge AI and Vision Alliance, Jan. 2024.   Accessed: Sep. 28, 2026. [Original source](https://www.edge-ai-vision.com/2024/01/what-is-a-uvc-camera-and-what-are-the-different-types-of-uvc-cameras/). (From RGB condensed research.)

[33] Edge AI and Vision Alliance, "Unlock Edge AI Potential: Why USB Cameras Are the Future of Vision Solutions," Edge AI and Vision Alliance, Nov. 2023.   Accessed: Sep. 28, 2026. [Original source](https://www.edge-ai-vision.com/2023/11/unlock-edge-ai-potential-why-usb-cameras-are-the-future-of-vision-solutions/). (From RGB condensed research.)

[34] Edge AI and Vision Alliance, "Choosing Best USB Camera for Edge AI Retail Systems," Edge AI and Vision Alliance, Dec. 2023.   Accessed: Sep. 28, 2026. [Original source](https://www.edge-ai-vision.com/2023/12/choosing-best-usb-camera-for-edge-ai-retail-systems/). (From RGB condensed research.)

[35] e-con Systems, "USB 3.0 vs. MIPI CSI-2: Choosing the Right Interface for High-Resolution Embedded Vision Systems," e-con Systems, 2023.   Accessed: Sep. 28, 2026. [Original source](https://www.e-consystems.com/blog/camera/technology-explained/usb-3-0-vs-mipi-csi-2-choosing-the-right-interface-for-high-resolution-embedded-vision-systems/). (From RGB condensed research.)

[36] OpenCV, "Camera Calibration and 3D Reconstruction (calib3d)," OpenCV Documentation.   Accessed: Sep. 28, 2026. [Original source](https://docs.opencv.org/5.0/tutorials/calib3d/camera_calibration/camera_calibration.html). (From RGB condensed research.)

[37] Intel, "Optimize Inference — OpenVINO™ Documentation," Intel Corporation, 2024.   Accessed: Sep. 28, 2026. [Original source](https://docs.openvino.ai/2026/openvino-workflow/running-inference/optimize-inference.html). (From RGB condensed research.)

[38] Intel, "Anomalib: An Anomaly Detection Library for Deep Learning," GitHub repository, 2024.   Accessed: Sep. 28, 2026. [Original source](https://github.com/openvinotoolkit/anomalib). (From RGB condensed research.)

[39] biscolab, "drone-defect-detector: Automated defect detection pipeline using YOLOv8," GitHub repository, 2024.   Accessed: Sep. 28, 2026. [Original source](https://github.com/biscolab/drone-defect-detector). (From RGB condensed research.)

[40] H. Ramezani, "drone-defect-detection: Structural surface defect classification with synthetic data," GitHub repository, 2024.   Accessed: Sep. 28, 2026. [Original source](https://github.com/HoomanRamezani/drone-defect-detection). (From RGB condensed research.)

[41] Lighting: Edmund Optics, Common Illumination Types. [Original source](https://www.edmundoptics.com/knowledge-center/application-notes/illumination/choose-the-correct-illumination/). (From Lighting condensed research.)

[42] Warm strip: Adafruit, Ultra Flexible White LED Strip, product 4613. [Original source](https://www.adafruit.com/product/4613). (From Lighting condensed research.)

[43] Nano: NVIDIA, Jetson Nano Developer Kit User Guide, December 2019. [Original source](https://developer.download.nvidia.com/embedded/L4T/r32-3-1_Release_v1.0/Jetson_Nano_Developer_Kit_User_Guide.pdf). (From Lighting condensed research.)

[44] Camera timing: Basler, Electronic Shutter Types. [Original source](https://docs.baslerweb.com/electronic-shutter-types). (From Lighting condensed research.)

[45] Thermal reflections: FLIR, How Does Emissivity Affect Thermal Imaging. [Original source](https://www.flir.com/en-asia/discover/professional-tools/how-does-emissivity-affect-thermal-imaging/). (From Lighting condensed research.)

[46] NVIDIA, Jetson Orin module specifications and Jetson Orin NX 16GB overview. [Original source](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/). (From ML condensed research.)

[47] NVIDIA, Jetson Orin module specifications and Jetson Orin NX 16GB overview. [Original source](https://developer.nvidia.com/blog/boost-edge-ai-performance-with-the-new-nvidia-jetson-orin-nx-16gb/). (From ML condensed research.)

[48] NVIDIA, DeepStream installation and platform/OS compatibility matrix. [Original source](https://docs.nvidia.com/deepstream/text/DS_Installation). (From ML condensed research.)

[49] C.-Y. Wang et al., Scaled-YOLOv4, 2020. [Original source](https://arxiv.org/abs/2011.08036). (From ML condensed research.)

[50] NVIDIA, JetPack SDK 6.2.3 and DeepStream SDK support matrix. [Original source](https://developer.nvidia.com/embedded/jetpack-sdk-623). (From ML condensed research.)

[51] NVIDIA, JetPack SDK 6.2.3 and DeepStream SDK support matrix. [Original source](https://catalog.ngc.nvidia.com/orgs/nvidia/-/collections/deepstream_sdk). (From ML condensed research.)

[52] Arducam, B0441 5 MP UVC camera. [Original source](https://www.arducam.com/5mp-autofocus-usb-camera-module-with-single-microphone-for-windows-linux-macos-android.html). (From ML condensed research.)

[53] GroupGets, FLIR Lepton 3.5 listing. [Original source](https://groupgets.com/collections/thermal-imaging). (From ML condensed research.)

[54] GroupGets, PureThermal 2 Shielded. [Original source](https://groupgets.com/products/purethermal-2-shielded). (From ML condensed research.)

[55] Samson, Go Mic specifications. [Original source](https://www.samsontech.com/products/microphones/usb-microphones/gomic/). (From ML condensed research.)

[56] MVTec, MVTec AD. [Original source](https://www.mvtec.com/research-teaching/datasets/mvtec-ad). (From ML condensed research.)

[57] MVTec, MVTec LOCO AD. [Original source](https://www.mvtec.com/research-teaching/datasets/mvtec-loco-ad). (From ML condensed research.)

[58] M. Batzner et al., EfficientAD: Accurate Visual Anomaly Detection at Millisecond-Level Latencies, WACV 2024. [Original source](https://openaccess.thecvf.com/content/WACV2024/html/Batzner_EfficientAD_Accurate_Visual_Anomaly_Detection_at_Millisecond-Level_Latencies_WACV_2024_paper.html). (From ML condensed research.)

[59] marcoslucianops, DeepStream-Yolo compatibility and configuration examples (third-party; pin and test a compatible revision, especially for newer DeepStream versions). [Original source](https://github.com/marcoslucianops/DeepStream-Yolo). (From ML condensed research.)

[60] Valinor, Dispatch specifications. [Original source](https://www.valinor.co/products/dispatch). (From Thermal condensed research.)

[61] Teledyne FLIR, Radiometry with Lepton; GroupGets, UVC capture and radiometry examples. [Original source](https://github.com/groupgets/purethermal1-uvc-capture). (From Thermal condensed research.)

[62] GroupGets, PureThermal Breakout Board. [Original source](https://groupgets.com/products/purethermal-breakout-board). (From Thermal condensed research.)

[63] ISO 17359:2018 (acoustic condition-monitoring background; as listed in acoustic research; online source not supplied).

[64] D. B. Rawat, C. Bajracharya, and G. Yan, "Edge AI-Enabled Anomaly Detection in Autonomous Systems via Multi-Modal Sensing," IEEE Internet of Things Journal, 2022 (as cited in acoustic research; bibliographic details not independently verified).

### 7.2 AI-assistance attribution

[65] Google Gemini, AI-assisted technical editing and formatting, 2026 (disclosed by the camera and acoustic research contributors; not a technical validation source).

[66] OpenAI, *ChatGPT (GPT-6)*, assisted in organizing the 2026 conceptual-design revisions, combining research citations, drafting proposed requirement/rebuttal wording, and document editing, November 18, 2026. Not an independent engineering validation source.

The team is responsible for confirming the correctness of all specifications, source links, and proposed decisions. Neither AI assistance nor the cited research papers establish tested performance on Valinor's drone or dock.

## 8. Statement of contributions

> **Member:** [Taylor, Lucas W.] 
> **Contribution statement:** Researched the AI/ML inspection architecture, including YOLOv4 component detection, NVIDIA Jetson Orin NX 16 GB deployment considerations, camera preprocessing, sensor fusion, JSON reporting, and preliminary model evaluation planning. Contributed to software integration and revision of the conceptual design.  
> **Date reviewed / signed by member:** [10/23/2026]

> **Member:** [Thongmanivong, Aaron N.]  
> **Contribution statement:** [Researched RGB camera and acoustic sensing options for the drone inspection system. Evaluated hardware capabilities, software compatibility, and potential methods for detecting visible defects and abnormal operating sounds. Contributed findings and recommendations to the conceptual design.]  
> **Date reviewed / signed by member:** [10/23/2026]

> **Member:** [Oke, Samuel I.]  
> **Contribution statement:** [Researched controlled lighting solutions for the drone inspection system, including LED selection, illumination requirements, and integration considerations. Evaluated how lighting conditions could affect image quality and the reliability of visual inspections.]  
> **Date reviewed / signed by member:** [10/23/2026]

> **Member:** [Lamberth, Michael S.]  
> **Contribution statement:** [Researched thermal imaging solutions for identifying abnormal temperatures in drone components. Evaluated thermal camera specifications, hardware integration requirements, and methods for incorporating temperature measurements into the inspection process.]  
> **Date reviewed / signed by member:** [10/23/2026]

> **Member:** [Heimermann, Benjamin C.]  
> **Contribution statement:** [Researched the existing drone docking system, including its hardware configuration and integration constraints. Investigated applicable IEEE and IEC standards related to electrical safety, system reliability, and hardware integration to help ensure the proposed system follows relevant engineering practices.]  
> **Date reviewed / signed by member:** [10/23/2026]
