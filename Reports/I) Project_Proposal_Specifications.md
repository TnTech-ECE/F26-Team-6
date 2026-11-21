
# F26_Team-6_Project_D.R.A.G.O.N.N.

## Introduction
As autonomous drones are deployed worldwide for logistics, infrastructure inspection, and public safety, keeping airframes reliable without human intervention has become a major challenge. Routine maintenance and early defect detection are essential to prevent crashes in the field. However, most current automated docking and inspection stations are proprietary, expensive, and locked down against third-party hardware or modifications. Existing industrial docks rely on high-cost vision hardware and continuous cloud processing pipelines. This creates a serious barrier for fleet operators and developers who need a flexible, modular system they can tailor to their specific airframes and operating budgets.

To address this challenge, Team 6 is developing Project D.R.A.G.O.N.N. (Drone Repair Analysis by Guided Optics via Neural Networks), an edge-based diagnostic payload built to integrate directly into Valinor Dispatch's autonomous drone docking stations for both stationary and mobile setups. The system uses commercial off-the-shelf (COTS) hardware, including a multi-angle UVC camera array and an independent radiometric thermal imager. Rather than building expensive custom software from scratch, each sensor pipeline is developed on top of proven open-source computer vision frameworks and public datasets. Data capture is run through an event-driven sequential loop, reading one device at a time so the USB bus does not choke and the edge board can dedicate all its processing power to the machine learning models.

The scope of this proposal outlines the operational background of automated UAV maintenance, the trade-offs among device hardware and communication protocols, and a comparative survey of existing commercial solutions. It establishes the theoretical foundation for using edge AI to detect structural and mechanical defects across machine vision, and radiometric thermal thresholding. Furthermore, the proposal details the technical specifications provided by Valinor Dispatch, evaluation criteria for successful field operation, budget allocations, available and required technical expertise, and a milestone timeline. Finally, it addresses the operational impact and broader industry implications of deploying an accessible diagnostic architecture built on open-source frameworks.

## Formulating the Problem

### Background
Autonomous drones are increasingly used in applications where the aircraft may operate repeatedly without a technician standing beside the launch site. Between flights, the drone may return to a fixed or mobile docking station for charging, data transfer, storage, and preparation for its next mission. Without a technician on site, remote operators need to know whether a required part appears to be missing, whether a component is unusually warm, and which areas still need inspection before the next flight. To assess how often a fault could go unnoticed, the team will introduce known faults during controlled testing. The proposed target is an overall PASS in no more than 5% of those inspections.

Preflight inspection is normally used to find visible damage, loose or missing hardware, contamination, abnormal motor condition, and other problems that could affect safe operation. Manufacturer guidance for aircraft such as the Skydio X10 includes examining items such as propellers, motors, airframe, and motor windings for damage or evidence of overheating. A human inspector can use sight, touch, and experience to judge these conditions. An unattended dock cannot reproduce that judgment directly, so the inspection problem must be converted into repeatable measurements that can be collected and evaluated automatically.

Remote cameras provide part of this capability, as commercial docking systems may allow an operator to view the aircraft while it is docked. However, a camera feed alone does not establish an automated health assessment. The usefulness of an image depends on camera placement, focus, lighting, reflections, working distance, and whether the required component is visible. A system that produces inconsistent images may report a difference caused by glare or docking position rather than actual damage. 

Within Valinor's dock, moving a camera farther from the aircraft brings more of the airframe into view, but each component occupies fewer pixels. A second viewing angle can help determine whether a propeller is missing or hidden behind an arm. Each additional view also adds data that the dock computer must receive and process. Capturing the views in sequence reduces peak demand on the shared USB connection, but leaves less time for the remaining checks before the next flight.

Capture order also affects how the team interprets thermal measurements. If thermal capture follows a series of visible images, adding another camera view delays the temperature reading. Motors cool after landing, and charging can change surface temperatures. A healthy motor measured immediately after one flight may therefore appear warmer than the same motor measured several minutes after another. To make those readings comparable, the team must account for elapsed time since landing and the aircraft's charging state when choosing the inspection sequence.

Furthermore, before passing any image to a machine learning model, the edge system must mathematically verify optical clarity to avoid false positives. This is done using an image processing quality gate, specifically calculating the Variance of the Laplacian to measure edge sharpness:

$$\text{Variance} = \frac{1}{N} \sum_{x,y} \left( \nabla^2 I(x,y) - \mu \right)^2$$

If this variance drops below a hardcoded threshold, the image is flagged as blurry, and the inspection is aborted before wasting edge compute resources. Therefore, the project aims to strictly control the image-capture conditions and verify image quality before applying machine-learning or anomaly-detection methods.

The prototype camera baseline is four Arducam B0471C enclosed, motorized-focus USB 3.0 cameras using Sony IMX519 16 MP sensors (`4656 × 3496`, 65° horizontal and 51° vertical field of view) and USB Video Class (UVC). The selected product listing specifies up to 80 fps at 720p; Arducam lists YUY2 capture up to 50 fps at 1080p and 8 fps at full 16 MP resolution. Its focus can be programmed from 10 cm to infinity, although the default range is 3 m to infinity, so close focus and focus control must be verified at the prototype's working distances. Each camera is assigned to an upper drone quadrant with overlap at the centerlines. The selected USBGear USBG-BREC304 four-port USB 3.2 Gen 1 powered hub carries the four RGB cameras; the FLIR PureThermal 3 interface uses a separate available host USB port. The IMX519 is the selected prototype camera, not a fixed long-term platform requirement: each camera shall connect through a replaceable module and documented interface so a newer or better camera can be adopted without redesigning the inspection system, provided it meets the coverage, focus, sampling, bandwidth, power, and host-compatibility requirements and passes recalibration and verification. Camera acquisition, sensor preprocessing, machine-learning inference, decision logic, storage, and dock communication remain modular so future hardware can be evaluated without rewriting the full system. [21][22]

## Preliminary Specifications
The following specifications define the initial problem boundaries. Numerical limits will be finalized after the team receives the drone geometry, dock dimensions, inspection-time requirement, environmental range, and acceptance criteria from the sponsor.

### System Capabilities
-	The system shall begin an inspection only after the dock confirms that the drone is present, correctly positioned, and no conflicting dock motion is active.
-	The system shall collect visible and thermal data using a repeatable inspection sequence.
-	The system shall associate each measurement with an inspection identifier, timestamp, sensor identifier, drone or asset identifier, and component region.
-	The system shall evaluate defined components individually rather than returning only one aircraft-wide result.
-	The system shall return PASS, WARN, FAIL, or INSUFFICIENT DATA for each required component or inspection region.
-	The system shall preserve the image, thermal region, or other evidence associated with a warning or failure.
-	The system shall report sensor, lighting, communication, storage, and processing faults separately from detected drone defects.
-	The system shall run routine inference locally without requiring a continuous internet connection.
### Visible Imaging and Controlled Lighting
-	The visible-camera subsystem shall use USB Video Class or another documented interface supported by the selected host. In this proposal, UVC refers to USB Video Class, not ultraviolet-C imaging [7].
-	The system shall provide repeatable coverage of all required inspection regions using fixed or mechanically repeatable camera mounts.
-	Camera intrinsics and lens distortion shall be calibrated, and calibration information shall be retained with the camera configuration [8].
-	Exposure, gain, focus, white balance, and other image settings shall be fixed after calibration unless the inspection procedure explicitly controls their adjustment.
-	The lighting subsystem shall support independently controlled illumination suitable for both general-condition imaging and defect-enhancement imaging.
-	The system shall verify that required lighting channels operate correctly before accepting an image for analysis.
-	The system shall detect unusable frames caused by blur, obstruction, saturation, insufficient illumination, missing data, or incorrect camera state.
-	Visible captures shall be synchronized with the inspection sequence so that dock motion, lighting transitions, or thermal capture do not corrupt the measurement.
- The camera subsystem shall achieve a target image sampling density of **5 pixels/mm** throughout each designated inspection region.
- The four-camera array shall cover the designated upper inspection regions from upper-corner mounts, with centerline overlap where needed to avoid blind spots; actual visibility and sampling shall be checked on the prototype airframe. [21]
- The camera subsystem shall support sequential capture through the selected four-port USBGear USBG-BREC304 hub. The four downstream USB-A ports are allocated to the RGB cameras; the thermal interface requires a separate available host USB port. Capture duration shall be measured against the overall inspection-cycle target. [22]
### Thermal Imaging
-	The thermal subsystem shall capture data with the visible inspection lights off unless testing demonstrates that the lighting system does not alter the required thermal measurement.
-	Thermal data shall be captured after a defined post-docking interval and shall include the elapsed time since landing or motor shutdown.
-	Each thermal record shall include available ambient temperature, camera status, calibration state, and operating-condition metadata.
-	The selected thermal camera and optical path shall provide sufficient field of view and pixels on each required component for the intended anomaly assessment.
-	If absolute temperature is required, the system shall use a radiometric camera and account for emissivity, reflected background temperature, viewing angle, optical-window transmission, and model-specific calibration requirements [9].
-	If only relative anomaly detection is required, the system shall compare equivalent components and validated healthy baselines under similar operating conditions.
-	The thermal camera shall be isolated from avoidable heat produced by processors, lighting drivers, power converters, chargers, fans, and dock airflow.
-	The system shall not accept a thermal result when the camera is unavailable, calibrating, obstructed, thermally unstable, or outside its approved operating condition.

### Data Processing and Decision Logic
-	Sensor acquisition, quality checks, preprocessing, model inference, result fusion, storage, and dock communication shall be implemented as separate functional stages.
-	The software shall support replacement of a camera, thermal module, inference model, or processing platform through documented interfaces or adapters.
-	The pipeline shall resize, normalize, register, or otherwise transform sensor data only according to the requirements of the selected method.
-	If pixel-level or feature-level visible-thermal fusion is used, the system shall include a validated spatial-registration method.
-	The visible-only inspection path shall remain testable independently of the thermal paths.
-	The system shall record inference time, end-to-end inspection time, dropped or invalid data, model version, configuration version, and processing-resource utilization.
-	A multimodal result shall identify which sensors contributed to the decision and shall not hide the loss or failure of one sensor.
-	Inspection thresholds shall be established from prototype testing with healthy units and representative defects rather than assumed in advance.
-	Trained models shall be versioned separately from the camera acquisition and dock-control software.
### Preliminary AI Pipeline Specifications
-	The following specifications are proposed for the AI subsystem and can be changed depending on the circumstances.
-	The AI pipeline shall accept frames produced by the UVC camera subsystem.
-	The AI pipeline shall accept frames or processed image data produced by the thermal imaging subsystem.
-	The system shall timestamp incoming frames so visible and thermal images can be paired as closely as practical. 
-	The pipeline shall include a method of resizing and normalizing images into the format expected by the selected neural network.
-	If pixel-level or feature-level RGB-T fusion is used, the pipeline shall include a method for spatially registering the thermal and visible images.
-	The AI model shall provide readable outputs such as object class, confidence level, bounding-box coordinates, and other necessary information.
-	The software shall record basic performance information such as inference time, frame rate, and dropped frames.
-	The AI pipeline shall be modular enough to allow different trained models to be tested without redesigning the image-acquisition subsystem.
-	The system should be capable of continuing operation if one imaging modality temporarily becomes unavailable.
-	The final inference system shall run locally on the selected processing hardware without requiring continuous internet access.
-	The AI system may include object tracking to reduce unstable detections between consecutive video frames.
-	The training process may occur on a more powerful desktop computer or cloud GPU. The final trained model will then be exported and deployed onto the embedded computer.

### Electrical Integration and Communications
-	Imaging, lighting, processing, and actuator loads shall use appropriately protected power branches. The selected USBGear USBG-BREC304 hub is supplied from 5 V/2 A (10 W); its four downstream ports support up to 900 mA each. Measure peak hub input power with all four cameras active and maintain at least 20% supply margin (no more than 8 W / 1.6 A at 5 V). Use the supplied adapter only where an approved AC source is available; otherwise use a protected, regulated 12 V-to-5 V converter rated for at least 2 A, with connector and polarity verified. [22]
-	Supply and return conductors shall be routed together to reduce loop area, and sensitive data or camera-power cables shall be separated from motors, solenoids, chargers, and other switching loads.
-	Cable shields and enclosure bonds shall be terminated using short, low-impedance connections consistent with the selected interface and equipment guidance.
-	The system shall use the cable type, maximum length, shielding, termination, and connector requirements specified for each communication interface.
-	Communication loss, dropped frames, device disconnection, undervoltage, and storage failure shall be detected and reported.
### Modularity and Maintainability
-	Cameras, lights, thermal modules, interface adapters, and processing hardware shall be replaceable without redesigning the entire inspection architecture.
-	Product-specific settings shall be stored in configuration files or device-adapter modules rather than distributed throughout the application.
-	The system shall support recalibration after a sensor, lens, mount, light, or drone model is changed.
-	The design shall provide service access for lens cleaning, diffuser cleaning, connector inspection, calibration targets, and replacement of high-wear components.
-	Documentation shall identify interfaces, pinouts, supply requirements, cable requirements, calibration data, software versions, and test procedures for each subsystem.
## Constraints
### Physical and Optical Constraints
-	Camera quantity, field of view, working distance, mounting angle, and lens selection are limited by the internal dock geometry and the dimensional envelope of the supported drones.
-	Required surfaces may be hidden by the landing gear, arms, payload, dock structure, or other components. The system cannot classify damage that is not represented in the captured data.
-	Docking-position variation can change image alignment, focus, reflections and thermal regions. Mechanical repeatability or validated image registration is required.
-	Ordinary visible-light glass or clear plastic may block long-wave infrared energy and cannot be placed in the thermal optical path unless its transmission has been verified.
- Camera hardware, including mounts, shall fit within the docking station's **30 x 40-inch footprint** and specified height envelope.
- Camera fields of view shall be selected to maintain the target sampling density of **5 pixels/mm** while covering their assigned inspection regions.
- Adjacent camera views shall minimize overlap where practical without creating unacceptable blind spots.
- Camera acquisition shall complete within the **10-second inspection-cycle limit**.
- Combined sensor data transfer shall remain within the measured usable USB bandwidth and available memory capacity.
- Camera data acquisition and processing shall not run continuously while the system is idle.
- Incremental idle power attributable to the cameras shall be measured before establishing a maximum allowable limit.
- The system shall not rely on USB suspension or selective-suspend behavior unless support is verified for the selected hardware and software.
### Processing and Interface Constraints
-	The number, resolution, format, and frame rate of connected sensors are limited by the host's communication bandwidth, memory, storage rate, and processing capability.
-	Multiple USB cameras may share a host controller. Simultaneous operation must remain within the measured capacity of the selected controller and hub.
-	Sequential capture can reduce simultaneous bandwidth demand, but the total inspection must still meet the sponsor's allowable turnaround time.
-	MIPI, SPI, I2S, and other board-level interfaces may require short cable lengths, compatible voltage levels, driver support, device-tree configuration, or a carrier board specific to the selected host.
-	A model that runs successfully on a development computer may exceed the memory, latency, power, or thermal limits of the deployed processor.
### Measurement and Dataset Constraints
-	A limited number of damaged drones or failure examples may be available. The project may require healthy-baseline anomaly detection, controlled defect samples, synthetic data, transfer learning, or a combination of these methods.
-	Public visible-thermal datasets may support early development but may not match the team's drone, camera geometry, dock lighting, thermal conditions, or target defect classes.
-	Thermal anomalies may indicate abnormal behavior without identifying the underlying cause. Final fault labels must be limited to what the test evidence supports.
-	False-positive and false-negative rates cannot be assumed from performance on unrelated industrial or public datasets.
-	PASS, WARN, and FAIL thresholds require validation on reserved data that is not used to train or tune the model.
### Environmental and Electrical Constraints
-	Ambient light leakage, dust, moisture, vibration, wind, rain, temperature variation, and contamination can change sensor measurements and must be addressed by the enclosure and inspection procedure.
-	Heat retained after flight or produced during charging may resemble a thermal fault unless capture timing and operating state are controlled.
-	Motors, solenoids, switching regulators, chargers, and lighting drivers may introduce conducted or radiated interference into camera, and processor circuits.
-	The available dock voltage, current capacity, grounding arrangement, connector ratings, and cable routes must be confirmed before final hardware selection.
### Safety and Operational Constraints
-	The inspection system shall not command propeller or motor operation unless the aircraft, enclosure, and dock controls provide an approved safe test condition.
-	A sensor or model failure shall not be interpreted as a healthy result. Missing or unreliable data shall produce an INSUFFICIENT DATA or equivalent fault state.
-	For flights under 14 CFR Part 107, the remote pilot in command must check the aircraft system before each flight to determine whether it is safe to operate [18]. The inspection report shall identify the components checked, flag incomplete checks, and provide supporting evidence for the pilot's assessment. A PASS applies only to the checks performed. The inspection system shall have no authority to release the aircraft for flight.
- Manufacturer-required maintenance procedures remain in effect.
-	Stored images, thermal data, identifiers, and maintenance records shall be protected according to the sponsor's access, retention, and cybersecurity requirements.
### Procurement and Product-Neutrality Constraints
-	Final component selection depends on verified availability, lead time, lifecycle status, environmental rating, exact interface support, documentation, and total integration cost.
-	Any procurement or compliance requirement, including NDAA-related requirements, must be confirmed for the exact camera, lens, interface board, and finished configuration rather than assumed from a product family.
-	The architecture shall not require a Jetson, Raspberry Pi, or any other single processing product unless the sponsor later establishes that product as a project requirement.
-	The first prototype may use convenient development hardware, but acceptance criteria shall describe required performance and interfaces rather than a brand name.

### Standards of Compliance
| Standard or specification | Effect on the proposed design |
| --- | --- |
| ISO 12233:2024 [10] | Use a resolution chart at the installed working distance to determine which features the camera and lens can resolve in each required region. Use those measurements to set the smallest feature the team will evaluate for detection. |
| ISO 15739:2023 [11] | Measure image noise under the proposed dock lighting and use the results to select exposure and gain settings for inspection. |
| USB Video Class 1.5 [7] | For each candidate USB camera, verify that the dock computer can acquire the required image format and resolution through its supported UVC interface. |

## Survey of Existing Solutions
Several existing approaches support equipment inspection through manual checks, remote cameras, visual anomaly detection, and thermal imaging. The proposed system will use USB Video Class (UVC) cameras to observe visible damage and a thermal imaging camera to identify abnormal heat signatures.

### Manual Drone Inspection and Skydio Dock for X10
Manufacturer procedures help identify components that should be examined before flight, such as examining motor windings for discoloration or evidence of heat damage. Skydio also describes embedded cameras in its Dock for X10 as supporting remote preflight vehicle inspection.
*   **Pros**: Dock cameras allow the aircraft to be observed remotely; manufacturer checklists help establish inspection targets; images support assessment without requiring a person beside the drone.
*   **Cons**: Camera inspection depends on unobstructed views; remote viewing does not itself establish automated damage detection; visible images do not quantify surface temperature.
*   **Gaps**: The cited descriptions do not demonstrate an automated inspection system that combines visible images and thermal data.
*   **Takeaways**: Define targets using manufacturer guidance, standardize imaging with consistent lighting and drone positioning, and preserve evidence by displaying the associated image region.

### Industrial Visual Anomaly Detection: PatchCore and Anomalib
PatchCore builds a representative memory bank of features from non-defective images, comparing new image regions with this reference to detect anomalies. Anomalib provides a PatchCore implementation and tools for training, evaluation, and inference, including OpenVINO support.
*   **Pros**: Reduces dependence on a comprehensive set of labeled defect classes; identifies suspicious regions; available software supports implementation.
*   **Cons**: An unusual appearance does not automatically identify a particular failure; industrial benchmark results do not establish accuracy on the selected aircraft; processing time and memory use require measurement.
*   **Gaps**: Representative drone images and independent defect examples are still required; a visible-image model does not establish temperature-based anomaly assessment.
*   **Takeaways**: Evaluate a candidate baseline against simpler image-processing methods, represent normal variation, test known defects using reserved test images, and validate thresholds.

### FLIR Thermal Imaging for Equipment Condition Monitoring
FLIR describes thermal imaging as supporting condition monitoring of motors, batteries, and electrical equipment. This provides a relevant precedent for examining heat patterns that may be difficult to recognize through visible inspection.
*   **Pros**: Surface heating can be examined without attaching sensors to every component; images locate hot regions; thermal measurements provide information beyond visible appearance.
*   **Cons**: Enclosed components may not be directly observable; expected post-flight heating must be distinguished from suspicious patterns; a thermal anomaly alone does not identify the underlying fault.
*   **Gaps**: Normal heat patterns must be established for the selected aircraft; general equipment examples do not supply validated drone temperature limits.
*   **Takeaways**: Control measurement timing, compare similar components, and validate separately before combining with visual results.

Published aircraft inspection studies illustrate two choices for Valinor's dock. Harras et al. reported 85.8% accuracy when classifying propeller damage in their study [19]. To evaluate whether damage is visible from the dock, the team would need labeled examples captured at the installed camera angles and under dock lighting. Rice et al. used hangar-mounted pan-tilt-zoom cameras to inspect selected regions of a parked aircraft [20]. In Valinor's smaller enclosure, fixed cameras must cover the required components from the available mounting positions. The proposed prototype will use those views to check component presence, compare surface temperatures in observable regions, and tell the operator which checks could not be completed.

## Measurements of Success
### Functionality
- The AI software shall reliably receive data from the imaging subsystem.
- The final implementation shall be capable of processing the camera stream continuously for at least one hour without software failure.
- The software shall identify and log dropped or invalid frames rather than silently failing.
- The RGB-only detector shall remain usable independently of the thermal pipeline.
- The camera subsystem shall demonstrate at least **5 pixels/mm** throughout designated inspection regions using a calibrated target.
- The camera subsystem shall complete at least **30 inspection trials** to evaluate acquisition time, failures, and cycle-time consistency.
- Each camera shall pass image-quality checks for sharpness, exposure, and frame completeness using validated acceptance thresholds.
- The acquisition software shall detect disconnected cameras and failed or invalid frames.
### Detection Accuracy
- All candidate models shall be evaluated using the same held-out test images.
- Precision, recall, and mean average precision (mAP) shall be recorded where applicable.
- RGB-only, thermal-only, and RGB-T models shall be evaluated separately so the effect of
fusion can be measured.
- A fusion model should only replace the simpler RGB baseline if it demonstrates a measurable advantage in conditions important to the project.
- Special test sets should include normal lighting as well as poor or unusual lighting conditions.
Processing Performance
- End-to-end AI latency shall be measured from acquisition of the frame through
generation of the final detection output.
- The fused AI pipeline should process at least 90% of the frame rate supplied by the slower imaging subsystem during continuous operation.
- CPU, GPU/NPU, RAM, and temperature utilization shall be recorded during extended
tests.
### Reliability
- Loss of the thermal input shall not require restarting the entire software stack if RGB-
only operation is possible.
- Invalid camera frames shall be ignored without terminating the application.
- The pipeline shall recover from temporary stream interruptions where practical.
### Modularity
- The AI model shall be stored separately from camera-acquisition code.
- A replacement model shall be capable of being installed without redesigning the complete
video pipeline.
- Camera preprocessing and model preprocessing shall be separated so changes in one subsystem do not unnecessarily affect the other.


## Resources
The prototype uses four Arducam B0471C enclosed, motorized-focus 16 MP IMX519 USB 3.0 UVC cameras for visible inspection and one FLIR Lepton 3.5 with a PureThermal 3 interface for thermal measurements. The RGB cameras connect to a USBGear USBG-BREC304 four-port powered hub; the thermal interface connects to a separate host USB port. These selected devices are a project baseline, with working-distance focus, USB bandwidth, and power margin to be verified during integration. [21][22]

The current design baseline reuses the existing NVIDIA Jetson Orin NX 16 GB as the inspection processor, subject to verification of the installed software stack and available resources. The project budget therefore assigns no incremental purchase cost to the compute unit. A separate desktop computer may be used for model development and training. Local storage will hold inspection evidence and logs; open-source machine-learning and image-processing tools will support development, subject to a license review before transfer.

### Budget Document

| Item | Estimated Price ($) |
| :--- | ---: |
| Four Arducam B0471C enclosed, motorized-focus 16 MP Sony IMX519 USB 3.0 UVC cameras ($125.99 each, listing snapshot) [21] | $503.96 |
| FLIR Lepton 3.5 and PureThermal 3 interface | $284.00 |
| Existing NVIDIA Jetson Orin NX 16 GB | $0 incremental |
| USBGear USBG-BREC304 four-port USB 3.2 Gen 1 powered hub [22] | $25.99 |
| LED strips, switching boards, lighting supply, diffusers, and fixture brackets | $111.80–$186.80 |
| Camera brackets, USB-C-to-Type-A data cables, wiring, and protected DC conversion | $70.00–$140.00 |
| **Mandatory prototype subtotal** | **$995.75–$1,140.75** |
| 15% planning reserve | $149.36–$171.11 |
| **Planning budget including reserve** | **$1,145.11–$1,311.86** |

The camera price is the $125.99-per-unit listing shown in the supplied product screenshot and is a price snapshot; confirm the live price and exact enclosed-camera variant at procurement. The hub amount is a planning estimate. Record actual order prices at procurement. The four hub ports serve the RGB cameras, and the PureThermal 3 interface requires a separate host USB port. Measure peak hub input power and retain at least 20% margin on its 5 V/2 A supply. The detailed BOM and assumptions govern the subsystem cost ranges; revise the estimate if supplier pricing or client-furnished equipment changes. [21][22]

## Personnel
Team 6 consist of members whose experience and expertise covers hardware integration, programming, and technical writing. Individual skills as follows:

*   **Ben Heimermann:** C/C++, 3d-modeling, Wiring and schematics
*   **Michael Lamberth:** C/C++, Python, Linux, Embedded Systems
*   **Samuel Oke:** C/C++, Wiring and Schematics, Hardware and Software Integration 
*   **Lucas Taylor:** C/C++, Python, Embedded Systems, Linux, Technical Documentation
*   **Aaron Thongmanivong:** C/C++, Python/MicroPython, Technical Writing, Embedded Systems

These combined qualifications are expected to be sufficient to test, process, and train models as needed. Team 6 aims to work hard to fill any gaps in expertise as needed to provide an optimal prototype.

The team has not assigned a directly chosen supervisor for the project, as the team has directed source of help and instructions from multiple professors for their specific field of study. These professors will be able to help in the development of the AI agent, machine learning pipeline system, and edge AI processing.

The Instructors are Dr. Christopher Johnson and Owen O'Connor, who will function as the quality assurance and control for the duration of the project. Major proposals and designs will be vetted and approved by the instructors.

## Stakeholders

The primary stakeholder for Project D.R.A.G.O.N.N. is Valinor Dispatch, the project sponsor. Valinor needs a way to check returning aircraft for missing components and unusual surface temperatures before the next flight. The system must fit within the existing dock infrastructure and provide results quickly enough to help reduce downtime during drone maintenance and repair.

Remote pilots and dock operators need a clear report identifying which components were inspected and which require further review. Maintenance technicians need the location of each suspected issue and the image or thermal measurement that prompted the warning. Valinor's clients depend on reliable service. An unnecessary warning can delay a scheduled mission, while a missed fault can leave an aircraft in service when it needs maintenance.

People near flight operations have an interest in the early detection of faults that could put them at risk. Limiting camera coverage to the aircraft and dock also helps protect their privacy. The Federal Aviation Administration (FAA) is a regulatory stakeholder because the inspection report supports the remote pilot in command's preflight assessment.

## Timeline
*   **Phase 1: Procurement & Architecture Setup** - Acquire UVC cameras, FLIR Lepton sensor, and edge compute hardware. Establish dock mounting constraints.
*   **Phase 2: Subsystem Orchestration** - Develop the Python-based sequential capture loop to prevent USB bus saturation and integrate the OpenCV Variance of the Laplacian blur check.
*   **Phase 3: Model Training** - Train visual anomaly models on pristine drone datasets (Anomalib) and calibrate thermal ROI thresholds.
*   **Phase 4: Fusion & Integration** - Merge optical and thermal metrics into the Diagnostic Consensus Engine to output structured JSON telemetry (PASS, WARN, FAIL).
*   **Phase 5: Field Testing** - Install the payload into the Valinor Dispatch dock and run full-cycle continuous tests to validate accuracy, latency, and reliability.

## Specific/Broad Implications

### Specific Implications
Affordability is the most impactful specific implication of Project D.R.A.G.O.N.N. Standard industrial vision systems are highly expensive, with basic camera arrays costing between $800 and $1,600. By utilizing widely available, standard USB camera modules, the project reduces this cost to roughly $250. This massive cost reduction frees up the budget for other critical sensors, like the thermal imager, and makes automated drone maintenance affordable for fleets of any size.

Furthermore, this setup is highly adaptable. Because the system relies on universally adopted USB connections rather than proprietary hardware, it completely avoids vendor lock-in. If a camera breaks or needs an upgrade, an operator can simply swap it out. The system automatically recognizes the new device without requiring complex software updates or custom coding, ensuring seamless long-term maintenance.

### Broader Implications, Ethics, and Responsibility as Engineers
As autonomous drones become common in logistics and public safety, mid-air mechanical failures pose a serious risk to people and property. Standard drone flight controllers cannot detect gradual physical wear. By using AI to catch mechanical issues"such as cracked propellers or failing motors"before a drone ever leaves the dock, this system actively prevents catastrophic crashes and improves public safety.

Because this is a safety-critical system, the engineering team has a strict ethical responsibility to ensure the AI never makes a guess based on bad data. To prevent false alarms or missed defects, the software runs automated quality checks before every inspection; for example, if a camera lens is blurry or dirty, the system immediately aborts the process rather than risking a bad assessment. Finally, the system guarantees transparency. It combines the visual and thermal results into a simple "PASS, WARN, or FAIL" report, ensuring that human operators always retain absolute clarity and final authority over the health of their fleet.

## References
[1] Skydio, "Skydio X10 Flight Checklist." [Online]. Available: https://support.skydio.com/hc/en-us/articles/25013300914203-Skydio-X10-Flight-Checklist

[2] Skydio, "The Future Of Scalable, Autonomous Flight And Data Capture Is Here With Dock For X10." [Online]. Available: https://www.skydio.com/blog/dock-for-x10-future-scalable-autonomous-flight-data-capture

[3] K. Roth et al., "Towards Total Recall in Industrial Anomaly Detection," Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 14318-14328, 2022. [Online]. Author manuscript available: https://arxiv.org/abs/2106.08265

[4] Anomalib maintainers, "Anomalib," GitHub repository. [Online]. Available: https://github.com/open-edge-platform/anomalib

[5] Anomalib maintainers, "PatchCore," Anomalib documentation. [Online]. Available: https://anomalib.readthedocs.io/en/latest/markdown/guides/reference/models/image/patchcore.html

[6] Teledyne FLIR, "Thermal Imaging for Data Centers," technical note. [Online]. Available: https://www.flir.com/contentassets/5f4aa9d1ef5a439da25b0c91333e83fe/flir-thermal-imaging-for-datacenters-tech-note.pdf

[7] USB Implementers Forum, "USB Device Class Definition for Video Devices, Version 1.5." [Online]. Available: https://www.usb.org/document-library/video-class-v15-document-set

[8] OpenCV, "Camera Calibration and 3D Reconstruction." [Online]. Available: https://docs.opencv.org/5.0/tutorials/calib3d/camera_calibration/camera_calibration.html

[9] Teledyne FLIR OEM, "Radiometry with Lepton." [Online]. Available: https://oem.flir.com/learn/thermal-integration-made-easy/radiometry-with-lepton/

[10] International Organization for Standardization, "ISO 12233:2024 - Digital Cameras - Resolution and Spatial Frequency Responses." [Online]. Available: https://www.iso.org/standard/88626.html

[11] International Organization for Standardization, "ISO 15739:2023 - Photography - Electronic Still-Picture Imaging - Noise Measurements." [Online]. Available: https://www.iso.org/standard/82233.html

[12] USB Implementers Forum, "USB Video Class v1.5 Specification." [Online]. Available: https://www.usb.org/document-library/video-class-v15-document-set

[15] International Electrotechnical Commission, "IEC 62368-1 - Audio/Video, Information and Communication Technology Equipment - Part 1: Safety Requirements." [Online]. Available: https://webstore.iec.ch/

[16] IEEE, "IEEE 7000-2021 - IEEE Standard Model Process for Addressing Ethical Concerns During System Design." [Online]. Available: https://standards.ieee.org/standard/7000-2021.html

[17] International Electrotechnical Commission, "IEC 62443-4-1:2018 - Security for Industrial Automation and Control Systems - Part 4-1: Secure Product Development Lifecycle Requirements." [Online]. Available: https://webstore.iec.ch/

[18] Electronic Code of Federal Regulations, "14 CFR § 107.15 - Condition for safe operation." [Online]. Available: https://www.ecfr.gov/current/title-14/chapter-I/subchapter-F/part-107/subpart-B/section-107.15

[19] M. S. Harras, S. Saleh, B. Battseren, and W. Hardt, "Vision-based Propeller Damage Inspection Using Machine Learning," Embedded Selforganising Systems, vol. 10, no. 7, 2023. [Online]. Available: https://www.bibliothek.tu-chemnitz.de/ojs/index.php/cs/article/view/604

[20] M. D. Rice et al., "Automated Multi-Camera Inspection System for Aircraft," Proceedings of the AAAI Conference on Artificial Intelligence, vol. 40, no. 48, 2026. [Online]. Available: https://doi.org/10.1609/aaai.v40i48.42378

[21] Arducam, *B0471 16 MP IMX519 USB 3.0 Camera Module Datasheet* and B0471/B0471C product information (motorized focus; B0471C enclosed variant). [Online]. Available: https://www.arducam.com/downloads/datasheet/B0471_16MP_IMX519_USB3.0_Camera_Datasheet.pdf and https://www.arducam.com/blog/product/arducam-16mp-imx519-motorized-focus-usb-3-0-camera-module/.

[22] USBGear, *USBG-BREC304 4-Port USB 3.2 Gen 1 Powered Hub*. [Online]. Available: https://www.usbgear.com/usbg-brec304.html.

Note: References [10] and [11] are official standard listings; full-text access may require purchase or Tennessee Tech library access. USB-IF specifications are available through its document library.

## Statement of Contributors
- Ben Heimermann: Resources 
- Michael Lamberth: Survey of Existing Solutions 
- Samuel Oke: Formulating the Problem, Preliminary Inspections, and Constraints
- Lucas Taylor: Preliminary Inspections and Measurement of Success
- Aaron Thongmanivong: Introduction, Formulating the Problem, Preliminary Specifications, Constraints, Resources, and Specific/Broad Implications