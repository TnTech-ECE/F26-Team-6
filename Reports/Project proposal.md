
# F26_Team-6_Project_D.R.A.G.O.N.N.

## Introduction
As autonomous drones are deployed worldwide for logistics, infrastructure inspection, and public safety, keeping airframes reliable without human intervention has become a major challenge. Routine maintenance and early defect detection are essential to prevent crashes in the field. However, most current automated docking and inspection stations are proprietary, expensive, and locked down against third-party hardware or modifications. Existing industrial docks rely on high-cost vision hardware and continuous cloud processing pipelines. This creates a serious barrier for fleet operators and developers who need a flexible, modular system they can tailor to their specific airframes and operating budgets.

To address this challenge, Team 6 is developing Project D.R.A.G.O.N.N. (Drone Repair Analysis by Guided Optics via Neural Networks), an edge-based diagnostic payload built to integrate directly into Valinor Dispatch’s autonomous drone docking stations for both stationary and mobile setups. The system uses commercial off-the-shelf (COTS) hardware, including a multi-angle UVC camera array, an independent radiometric thermal imager, and a dedicated USB microphone. Rather than building expensive custom software from scratch, each sensor pipeline is developed on top of proven open-source computer vision frameworks and public datasets. Data capture is run through an event-driven sequential loop, reading one device at a time so the USB bus does not choke and the edge board can dedicate all its processing power to the machine learning models.

The scope of this proposal outlines the operational background of automated UAV maintenance, the trade-offs among device hardware and communication protocols, and a comparative survey of existing commercial solutions. It establishes the theoretical foundation for using edge AI to detect structural and mechanical defects across machine vision, acoustic analysis, and radiometric thermal thresholding. Furthermore, the proposal details the technical specifications provided by Valinor Dispatch, evaluation criteria for successful field operation, budget allocations, available and required technical expertise, and a milestone timeline. Finally, it addresses the operational impact and broader industry implications of deploying an accessible diagnostic architecture built on open-source frameworks.

## Formulating the Problem

### Background
Autonomous drones are increasingly used in applications where the aircraft may operate repeatedly without a technician standing beside the launch site. Between flights, the drone may return to a fixed or mobile docking station for charging, data transfer, storage, and preparation for its next mission. This operating model reduces the amount of direct human supervision, but it also creates a maintenance problem: damage or gradual wear must be identified before the aircraft is released for another flight.

Preflight inspection is normally used to find visible damage, loose or missing hardware, contamination, abnormal motor condition, and other problems that could affect safe operation. Manufacturer guidance for aircraft such as the Skydio X10 includes examining items such as propellers, motors, airframe, and motor windings for damage or evidence of overheating. A human inspector can use sight, touch, sound, and experience to judge these conditions. An unattended dock cannot reproduce that judgment directly, so the inspection problem must be converted into repeatable measurements that can be collected and evaluated automatically.

Remote cameras provide part of this capability, as commercial docking systems may allow an operator to view the aircraft while it is docked. However, a camera feed alone does not establish an automated health assessment. The usefulness of an image depends on camera placement, focus, lighting, reflections, working distance, and whether the required component is visible. A system that produces inconsistent images may report a difference caused by glare or docking position rather than actual damage. 

Furthermore, before passing any image to a machine learning model, the edge system must mathematically verify optical clarity to avoid false positives. This is done using an image processing quality gate, specifically calculating the Variance of the Laplacian to measure edge sharpness:

$$\text{Variance} = \frac{1}{N} \sum_{x,y} \left( \nabla^2 I(x,y) - \mu \right)^2$$

If this variance drops below a hardcoded threshold, the image is flagged as blurry, and the inspection is aborted before wasting edge compute resources. Therefore, the project aims to strictly control the image-capture conditions and verify image quality before applying machine-learning or anomaly-detection methods.

The project is intended to remain portable across future drone models and computing platforms. The first prototype may use a particular embedded computer, camera, or microphone, but the architecture should not require one vendor's product to remain functional. USB Video Class cameras, standard audio interfaces, Ethernet, MIPI, SPI, or supported adapter boards can be used where appropriate. Camera acquisition, sensor preprocessing, machine-learning inference, decision logic, storage, and dock communication are designed to remain separate enough that one layer can be changed without rewriting the full system.

## Preliminary Specifications
The following specifications define the initial problem boundaries. Numerical limits will be finalized after the team receives the drone geometry, dock dimensions, inspection-time requirement, environmental range, and acceptance criteria from the sponsor.

### System Capabilities
*   The system should begin an inspection only after the dock confirms that the drone is present, correctly positioned, and no conflicting dock motion is active.
*   The system aims to collect visible, thermal, and approved acoustic data using a repeatable inspection sequence.
*   The system is expected to associate each measurement with an inspection identifier, timestamp, sensor identifier, drone or asset identifier, and component region.
*   The system should evaluate defined components individually rather than returning only one aircraft-wide result.
*   The system is designed to return PASS, WARN, FAIL, or INSUFFICIENT DATA for each required component or inspection region.
*   The system should preserve the image, thermal region, waveform, spectrum, or other evidence associated with a warning or failure.
*   The system will report sensor, lighting, communication, storage, and processing faults separately from detected drone defects.
*   The system aims to run routine inference locally without requiring a continuous internet connection.

### Visible Imaging and Controlled Lighting
*   The visible-camera subsystem plans to use USB Video Class or another documented interface supported by the selected host.
*   The system should provide repeatable coverage of all required inspection regions using fixed or mechanically repeatable camera mounts.
*   Camera intrinsics and lens distortion are expected to be calibrated, and calibration information will be retained with the camera configuration.
*   Exposure, gain, focus, white balance, and other image settings should remain fixed after calibration unless the inspection procedure explicitly controls their adjustment.
*   The lighting subsystem is designed to support independently controlled illumination suitable for both general-condition imaging and defect-enhancement imaging.
*   The system will verify that required lighting channels operate correctly before accepting an image for analysis.
*   The system aims to detect unusable frames caused by blur, obstruction, saturation, insufficient illumination, missing data, or incorrect camera state.
*   Visible captures should be synchronized with the inspection sequence so that dock motion, lighting transitions, or thermal captures do not corrupt the measurement.

### Thermal Imaging
*   The thermal subsystem should capture data with the visible inspection lights off unless testing demonstrates that the lighting system does not alter the required thermal measurement.
*   Thermal data is expected to be captured after a defined post-docking interval and should include the elapsed time since landing or motor shutdown.
*   Each thermal record should include available ambient temperature, camera status, calibration state, and operating-condition metadata.
*   The selected thermal camera and optical path aim to provide a sufficient field of view and pixels on each required component for the intended anomaly assessment.
*   If absolute temperature is required, the system will use a radiometric camera and account for emissivity, reflected background temperature, viewing angle, optical-window transmission, and model-specific calibration requirements.
*   If only relative anomaly detection is required, the system plans to compare equivalent components and validated healthy baselines under similar operating conditions.
*   The thermal camera should be isolated from avoidable heat produced by processors, lighting drivers, power converters, chargers, fans, and dock airflow.
*   The system is configured to reject a thermal result when the camera is unavailable, calibrating, obstructed, thermally unstable, or outside its approved operating condition.

### Acoustic Inspection
*   The acoustic subsystem aims to record a repeatable audio interval using a documented microphone interface and sampling configuration.
*   Any motor-spin or powered acoustic test will occur only when permitted by the drone manufacturer, sponsor, dock safety controls, and project test procedure.
*   The recording should include the commanded operating state, sample rate, duration, microphone identifier, and relevant dock or environmental conditions.
*   The system will retain either the original recording or an approved processed representation sufficient for later review and model validation.
*   Acoustic preprocessing may include filtering, normalization, time-frequency analysis, spectral features, or learned features produced by the selected model.
*   The system is expected to reject or flag recordings affected by clipping, missing samples, excessive background noise, dock actuator noise, or an incorrect motor command.
*   Acoustic results should remain identifiable as a separate sensor output before they are combined with visible or thermal evidence.

### Data Processing and Decision Logic
*   Sensor acquisition, quality checks, preprocessing, model inference, result fusion, storage, and dock communication are implemented as separate functional stages.
*   The software is designed to support the replacement of a camera, microphone, thermal module, inference model, or processing platform through documented interfaces or adapters.
*   The pipeline should resize, normalize, register, or otherwise transform sensor data only according to the requirements of the selected method.
*   If pixel-level or feature-level visible-thermal fusion is used, the system aims to include a validated spatial-registration method.
*   The visible-only inspection path is expected to remain testable independently of the thermal and acoustic paths.
*   The system will record inference time, end-to-end inspection time, dropped or invalid data, model version, configuration version, and processing-resource utilization.
*   A multimodal result should identify which sensors contributed to the decision and avoid hiding the loss or failure of one sensor.
*   Inspection thresholds are expected to be established from prototype testing with healthy units and representative defects rather than assumed in advance.
*   Trained models will be versioned separately from the camera acquisition and dock-control software.

### Preliminary AI Pipeline Specifications
*   The AI pipeline should accept frames produced by the UVC camera subsystem.
*   The AI pipeline should accept frames or processed image data produced by the thermal imaging subsystem.
*   The system aims to timestamp incoming frames so visible and thermal images can be paired as closely as practical.
*   The pipeline will include a method of resizing and normalizing images into the format expected by the selected neural network.
*   If pixel-level or feature-level RGB-T fusion is used, the pipeline aims to include a method for spatially registering the thermal and visible images.
*   The AI model is expected to provide readable outputs such as object class, confidence level, bounding-box coordinates, and other necessary information.
*   The software will record basic performance information such as inference time, frame rate, and dropped frames.
*   The AI pipeline is designed to be modular enough to allow different trained models to be tested without redesigning the image-acquisition subsystem.
*   The system should be capable of continuing operation if one imaging modality temporarily becomes unavailable.
*   The final inference system plans to run locally on the selected processing hardware without requiring continuous internet access.
*   The AI system may include object tracking to reduce unstable detections between consecutive video frames.
*   The training process may occur on a more powerful desktop computer or cloud GPU, and the final trained model will then be exported and deployed onto the embedded computer.

### Electrical Integration and Communications
*   Imaging, lighting, processing, and actuator loads should use appropriately protected power branches.
*   Continuous, startup, strobe, calibration, and simultaneous peak loads are expected to be included in the power budget.
*   Supply and return conductors should be routed together to reduce loop area, and sensitive data or camera-power cables should be separated from motors, solenoids, chargers, and other switching loads.
*   Inductive loads will include manufacturer-approved flyback, transient-voltage suppression, or snubber protection as appropriate.
*   Cable shields and enclosure bonds should be terminated using short, low-impedance connections consistent with the selected interface and equipment guidance.
*   The system plans to use the cable type, maximum length, shielding, termination, and connector requirements specified for each communication interface.
*   Communication loss, dropped frames, device disconnection, undervoltage, and storage failure should be detected and reported.
*   The dock interface is expected to receive a structured inspection result containing component identifiers, sensor status, evidence locations, decision state, and relevant confidence or anomaly values.

### Modularity and Maintainability
*   Cameras, lights, microphone assemblies, thermal modules, interface adapters, and processing hardware are designed to be replaceable without redesigning the entire inspection architecture.
*   Product-specific settings should be stored in configuration files or device-adapter modules rather than distributed throughout the application.
*   The system plans to support recalibration after a sensor, lens, mount, light, or drone model is changed.
*   The design aims to provide service access for lens cleaning, diffuser cleaning, connector inspection, calibration targets, and replacement of high-wear components.
*   Documentation will identify interfaces, pinouts, supply requirements, cable requirements, calibration data, software versions, and test procedures for each subsystem.

### Constraints
*   **Physical and Optical Constraints**: Camera quantity, field of view, working distance, mounting angle, and lens selection are limited by the internal dock geometry and the dimensional envelope of the supported drones. Required surfaces may be hidden by landing gear or payloads, and docking-position variation can change image alignment. Ordinary visible-light glass or clear plastic may block long-wave infrared energy and cannot be placed in the thermal optical path unless its transmission has been verified.
*   **Processing and Interface Constraints**: The number, resolution, format, and frame rate of connected sensors are limited by the host's communication bandwidth, memory, storage rate, and processing capability. Multiple USB cameras and a USB microphone may share a host controller; simultaneous operation must remain within the measured capacity. Sequential capture can reduce simultaneous bandwidth demand.
*   **Measurement and Dataset Constraints**: A limited number of damaged drones or failure examples may be available. Public visible-thermal or acoustic datasets may support early development but may not match the team's drone, camera geometry, or target defect classes. Thermal and acoustic anomalies may indicate abnormal behavior without identifying the underlying cause. PASS, WARN, and FAIL thresholds require validation on reserved data.
*   **Environmental and Electrical Constraints**: Ambient light leakage, dust, moisture, vibration, wind, rain, temperature variation, and contamination can change sensor measurements. Heat retained after flight or produced during charging may resemble a thermal fault unless capture timing and operating state are controlled.
*   **Safety and Operational Constraints**: The inspection system will avoid commanding propeller or motor operation unless the aircraft, enclosure, and dock controls provide an approved safe test condition. A sensor or model failure cannot be interpreted as a healthy result. The inspection result is intended to support, not replace, manufacturer-required maintenance and human operational authority.
*   **Procurement and Product-Neutrality Constraints**: Final component selection depends on verified availability, lead time, lifecycle status, environmental rating, exact interface support, documentation, and total integration cost. The architecture does not inherently require a Jetson, Raspberry Pi, or any other single processing product unless the sponsor later establishes that product as a project requirement.

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

## Measurements of Success
### Functionality
*   The AI software needs to reliably receive data from the imaging subsystem.
*   The final implementation should be capable of processing the camera stream continuously for at least one hour without software failure.
*   The software aims to identify and log dropped or invalid frames rather than silently failing.
*   The RGB-only detector is expected to remain usable independently of the thermal pipeline.

### Detection Accuracy
*   All candidate models should be evaluated using the same held-out test images.
*   Precision, recall, and mean average precision (mAP) will be recorded where applicable.
*   RGB-only, thermal-only, and RGB-T models are expected to be evaluated separately so the effect of fusion can be measured.
*   A fusion model should only replace the simpler RGB baseline if it demonstrates a measurable advantage in conditions important to the project.
*   Special test sets plan to include normal lighting as well as poor or unusual lighting conditions.

### Processing Performance
*   End-to-end AI latency aims to be measured from acquisition of the frame through generation of the final detection output.
*   The fused AI pipeline should process at least 90% of the frame rate supplied by the slower imaging subsystem during continuous operation.
*   CPU, GPU/NPU, RAM, and temperature utilization will be recorded during extended tests.

### Reliability
*   Loss of the thermal input should not require restarting the entire software stack if RGB-only operation is possible.
*   Invalid camera frames are expected to be ignored without terminating the application.
*   The pipeline plans to recover from temporary stream interruptions where practical.

### Modularity
*   The AI model will be stored separately from camera-acquisition code.
*   A replacement model should be capable of being installed without redesigning the complete video pipeline.
*   Camera preprocessing and model preprocessing are intended to be separated so changes in one subsystem do not unnecessarily affect the other.

## Resources
The proposed system requires a combination of both commercial hardware and open-source software to accomplish the automated drone analysis. The main hardware required includes a UVC camera array to detect visual flaws, a thermal imaging camera to detect temperature anomalies, and a USB microphone for auditory analysis, especially for the propellors. These sensors complement each other to give a comprehensive diagnostic analysis of the docked drone.

A suitable board will be used to perform image processing, machine-learning, and analysis of sensor data. A separate, more powerful, desktop computer may also be used for AI development and training before it is deployed to the main system. Adequate storage will also be required to keep sensor data, training datasets, and flags. Open-source machine learning and image processing software will be used to develop and train the AI models, and publicly available datasets may also be used to supplement data collected during testing.

### Budget Document

| Item | Estimated Price ($) |
| :--- | :--- |
| Quad-Camera UVC Array (8MP–12MP CS-Mount) | $250 |
| Thermal Imager (FLIR Lepton 3.5 & PureThermal Board) | $300 |
| Edge Compute Unit | $500 |
| Standard USB Audio Class Microphone | $50 |
| Powered USB 3.0 Hub (for sequential data orchestration) | $30 |
| Controlled Lighting (COB LED strips, diffusers, polarizers) | $100 |
| Mounting Hardware, Enclosures, & Wiring | $150 |
| **Total** | **$1,380** |

## Personnel
Team 6 consist of members whose experience and expertise covers hardware integration, programming, and technical writing. Individual skills as follows:

*   **Ben Heimermann:** C/C++, 
*   **Michael Lamberth:** C/C++, 
*   **Samuel Oke:** C/C++, 
*   **Lucas Taylor:** C/C++, Python, Embedded Systems, Linux, Technical Documentation
*   **Aaron Thongmanivong:** C/C++, Python/MicroPython, Technical Writing, Embedded Systems

These combined qualifications are expected to be sufficient to test, process, and train models as needed. Team 6 aims to work hard to fill any gaps in expertise as needed to provide an optimal prototype.

The team has not assigned a directly chosen supervisor for the project, as the team has directed source of help and instructions from multiple professors for their specific field of study. These professors will be able to help in the development of the AI agent, machine learning pipeline system, and edge AI processing.

The Instructors are Dr. Christopher Johnson and Owen O'Connor, who will function as the quality assurance and control for the duration of the project. Major proposals and designs will be vetted and approved by the instructors.

The project is being developed in direct partnership with our primary client, Valinor Dispatch, to ensure the payload conforms to the physical and software constraints of their existing dock infrastructure.

## Timeline
*   **Phase 1: Procurement & Architecture Setup** – Acquire UVC cameras, FLIR Lepton sensor, USB microphone, and edge compute hardware. Establish dock mounting constraints.
*   **Phase 2: Subsystem Orchestration** – Develop the Python-based sequential capture loop to prevent USB bus saturation and integrate the OpenCV Variance of the Laplacian blur check.
*   **Phase 3: Model Training** – Train visual anomaly models on pristine drone datasets (Anomalib), calibrate thermal ROI thresholds, and validate the 1D-CNN acoustic model on low-RPM motor data.
*   **Phase 4: Fusion & Integration** – Merge optical, thermal, and acoustic metrics into the Diagnostic Consensus Engine to output structured JSON telemetry (PASS, WARN, FAIL).
*   **Phase 5: Field Testing** – Install the payload into the Valinor Dispatch dock and run full-cycle continuous tests to validate accuracy, latency, and reliability.

## Specific/Broad Implications

### Specific Implications
Affordability is the most impactful specific implication of Project D.R.A.G.O.N.N. Standard industrial vision systems are highly expensive, with basic camera arrays costing between $800 and $1,600. By utilizing widely available, standard USB camera modules, the project reduces this cost to roughly $250. This massive cost reduction frees up the budget for other critical sensors, like the thermal imager, and makes automated drone maintenance affordable for fleets of any size.

Furthermore, this setup is highly adaptable. Because the system relies on universally adopted USB connections rather than proprietary hardware, it completely avoids vendor lock-in. If a camera breaks or needs an upgrade, an operator can simply swap it out. The system automatically recognizes the new device without requiring complex software updates or custom coding, ensuring seamless long-term maintenance.

### Broader Implications, Ethics, and Responsibility as Engineers
As autonomous drones become common in logistics and public safety, mid-air mechanical failures pose a serious risk to people and property. Standard drone flight controllers cannot detect gradual physical wear. By using AI to catch mechanical issues—such as cracked propellers or failing motors—before a drone ever leaves the dock, this system actively prevents catastrophic crashes and improves public safety.

Because this is a safety-critical system, the engineering team has a strict ethical responsibility to ensure the AI never makes a guess based on bad data. To prevent false alarms or missed defects, the software runs automated quality checks before every inspection; for example, if a camera lens is blurry or dirty, the system immediately aborts the process rather than risking a bad assessment. Finally, the system guarantees transparency. It merges the visual, thermal, and audio data into a simple "PASS, WARN, or FAIL" report, ensuring that human operators always retain absolute clarity and final authority over the health of their fleet.

## References
[1] Skydio, “Skydio X10 Flight Checklist.” [Online]. Available: https://support.skydio.com/hc/en-us/articles/25013300914203-Skydio-X10-Flight-Checklist

[2] Skydio, “The Future Of Scalable, Autonomous Flight And Data Capture Is Here With Dock For X10.” [Online]. Available: https://www.skydio.com/blog/dock-for-x10-future-scalable-autonomous-flight-data-capture

[3] K. Roth et al., “Towards Total Recall in Industrial Anomaly Detection,” Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pp. 14318–14328, 2022. [Online]. Author manuscript available: https://arxiv.org/abs/2106.08265

[4] Anomalib maintainers, “Anomalib,” GitHub repository. [Online]. Available: https://github.com/open-edge-platform/anomalib

[5] Anomalib maintainers, “PatchCore,” Anomalib documentation. [Online]. Available: https://anomalib.readthedocs.io/en/latest/markdown/guides/reference/models/image/patchcore.html

[6] Teledyne FLIR, “Thermal Imaging for Data Centers,” technical note. [Online]. Available: https://www.flir.com/contentassets/5f4aa9d1ef5a439da25b0c91333e83fe/flir-thermal-imaging-for-datacenters-tech-note.pdf

[7] USB Implementers Forum, “USB Device Class Definition for Video Devices, Version 1.5.” [Online]. Available: https://www.usb.org/document-library/video-class-v15-document-set

[8] OpenCV, “Camera Calibration and 3D Reconstruction.” [Online]. Available: https://docs.opencv.org/5.0/tutorials/calib3d/camera_calibration/camera_calibration.html

[9] Teledyne FLIR OEM, “Radiometry with Lepton.” [Online]. Available: https://oem.flir.com/learn/thermal-integration-made-easy/radiometry-with-lepton/
