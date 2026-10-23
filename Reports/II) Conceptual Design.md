# F26 Team 6 Project D.R.A.G.O.N.N Conceptual Design

## Introduction
&nbsp;&nbsp;&nbsp; With the increasingly deployment of autonomous drones for logistics, infrastructure inspection, and public safety, the health assessment of the drone has been an interesting topic for many manufactures to reduce drone down-time and unnecessary human intervention. In which routine inspections for early defect detection is crucial to prevent malfunctions in the field and help find common problems. However, most drone docking stations that have an autonomous inspection function are expensive, has proprietary equipment, and locked down against third-part hardware or mods. Also, the use of high-cost vision hardware and continuous cloud processing pipelines make it a constraining choice. This creates a serious barrier for fleet operators and developers who need a flexible, modular system they can tailor to their specific airframes and operating budgets.

&nbsp;&nbsp;&nbsp; To address this challenge, Team 6 is developing Project D.R.A.G.O.N.N. (Drone Repair Analysis by Guided Optics via Neural Networks), an edge-based diagnostic payload built to integrate directly into Valinor Dispatch’s autonomous drone docking stations for both stationary and mobile setups. The system uses commercial off-the-shelf (COTS) hardware, including a multi-angle UVC camera array, an independent radiometric thermal imager, and a dedicated USB microphone. Rather than building expensive custom software from scratch, each sensor pipeline is developed on top of proven open-source computer vision frameworks and public datasets. Data capture is run through an event-driven sequential loop, reading one device at a time so the USB bus does not choke, and the edge board can dedicate all its processing power to the machine learning models.

&nbsp;&nbsp;&nbsp; This document outlines the objectives of the system design alongside the constraints and specifications defined by Valinor Dispatch (client), project advisors, and Team 6. The document begins with a concise problem formulation that identifies critical constraints imposed by the operating environment and client requirements. Next, a comparative analysis of potential subsystem solutions is presented, followed by the selected high-level architecture designed to minimize operational risk and maximize stakeholder value. Finally, the document provides a comprehensive breakdown of each atomic subsystem to detail hardware, software, and physical integration across the complete platform.

&nbsp;&nbsp;&nbsp; This specification delivers a comprehensive conceptual design centered on clarity, modularity, and technical rigor. Detailed hardware block diagrams, operational flowcharts, and formal "shall" requirements define each subsystem to ensure all functional blocks remain independent, measurable, and testable. 

&nbsp;&nbsp;&nbsp; Furthermore, the document incorporates professional ethics and applicable industry standards (including IEEE, ISO, and USB-IF specifications) into the system architecture. Finally, a refined project plan—featuring a detailed Bill of Materials, task allocation, and a milestone Gantt chart—provides the verified framework required to advance into detailed design and hardware fabrication.

## Restating the Fully Formulated Problem
&nbsp;&nbsp;&nbsp; As stated in Team 6's project proposal, autonomous drones are increasingly used in applications where the aircraft may operate repeatedly without a technician standing beside the launch site [1]. During the downtime of the drones, it returns to a docking station in which the drone can charge, offload any necessary data, be put in storage, or be inspected. However, to minimize the need for human inspection there is a need to make this process autonomous while maintaining a level of performance of a human.

&nbsp;&nbsp;&nbsp; Existing inspection solutions come with a large cost-prohibitive, hardware proprietary, and software locked build. Specifically, for a fleet of drones that consist of different types will need different measurements, possibly different camera setup, or be ported to a larger docking station.

&nbsp;&nbsp;&nbsp; Team 6 proposes the design and development of a lower-cost, modular/repairable, adjustable, and flexible system that allows for different drones to be analyzed in different environments. The fully formulated object is to deliver an affordable (minimize cost) and replicable prototype for Valinor Dispatch that integrates machine vision, thermal machine vision, an optional acoustic device, and lighting solution. In which the theoretical environment will be largest docking station specified by Valinor Dispatch.

For each subsystem, team 6 has placed specifications, requirements, and constraints on the solution due to the client standards or needs, in-depth analysis of solutions, and standardized compliances.

### System Capabilities
-	The system shall begin an inspection only after the dock confirms that the drone is present, correctly positioned, and no conflicting dock motion is active.
-	The system shall collect visible, thermal, and approved acoustic data using a repeatable inspection sequence.
-	The system shall associate each measurement with an inspection identifier, timestamp, sensor identifier, drone or asset identifier, and component region.
-	The system shall evaluate defined components individually rather than returning only one aircraft-wide result.
-	The system shall return PASS, WARN, FAIL, or INSUFFICIENT DATA for each required component or inspection region.
-	The system shall preserve the image, thermal region, waveform, spectrum, or other evidence associated with a warning or failure.
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

### Thermal Imaging
-	The thermal subsystem shall capture data with the visible inspection lights off unless testing demonstrates that the lighting system does not alter the required thermal measurement.
-	Thermal data shall be captured after a defined post-docking interval and shall include the elapsed time since landing or motor shutdown.
-	Each thermal record shall include available ambient temperature, camera status, calibration state, and operating-condition metadata.
-	The selected thermal camera and optical path shall provide sufficient field of view and pixels on each required component for the intended anomaly assessment.
-	If absolute temperature is required, the system shall use a radiometric camera and account for emissivity, reflected background temperature, viewing angle, optical-window transmission, and model-specific calibration requirements [9].
-	If only relative anomaly detection is required, the system shall compare equivalent components and validated healthy baselines under similar operating conditions.
-	The thermal camera shall be isolated from avoidable heat produced by processors, lighting drivers, power converters, chargers, fans, and dock airflow.
-	The system shall not accept a thermal result when the camera is unavailable, calibrating, obstructed, thermally unstable, or outside its approved operating condition.

### Acoustic Inspection
-	The acoustic subsystem shall record a repeatable audio interval using a documented microphone interface and sampling configuration.
-	Any motor-spin or powered acoustic test shall occur only when permitted by the drone manufacturer, sponsor, dock safety controls, and project test procedure.
-	The recording shall include the commanded operating state, sample rate, duration, microphone identifier, and relevant dock or environmental conditions.
-	The system shall retain either the original recording or an approved processed representation sufficient for later review and model validation.
-	Acoustic preprocessing may include filtering, normalization, time-frequency analysis, spectral features, or learned features produced by the selected model.
-	The system shall reject or flag recordings affected by clipping, missing samples, excessive background noise, dock actuator noise, or an incorrect motor command.
-	Acoustic results shall remain identifiable as a separate sensor output before they are combined with visible or thermal evidence.
Data Processing and Decision Logic
-	Sensor acquisition, quality checks, preprocessing, model inference, result fusion, storage, and dock communication shall be implemented as separate functional stages.
-	The software shall support replacement of a camera, microphone, thermal module, inference model, or processing platform through documented interfaces or adapters.
-	The pipeline shall resize, normalize, register, or otherwise transform sensor data only according to the requirements of the selected method.
-	If pixel-level or feature-level visible-thermal fusion is used, the system shall include a validated spatial-registration method.
-	The visible-only inspection path shall remain testable independently of the thermal and acoustic paths.
-	The system shall record inference time, end-to-end inspection time, dropped or invalid data, model version, configuration version, and processing-resource utilization.
-	A multimodal result shall identify which sensors contributed to the decision and shall not hide the loss or failure of one sensor.
-	Inspection thresholds shall be established from prototype testing with healthy units and representative defects rather than assumed in advance.
-	Trained models shall be versioned separately from the camera acquisition and dock-control software.

<br>

## Comparitive Analysis of Potential Solutions 


<br>

## High-Level Solution


<br>

## Atomic Subsystems Specifications


<br>

## Ethical, Professional, and Standards Considerations


<br>

## Resources

### Budget

### Division of Labor

### Timeline


<br>

## References
[1] TnTech ECE F26 Team 6, "I) Project_Proposal_Specifications," GitHub repository, 12 Oct. 2026. [Online]. Available: https://github.com/TnTech-ECE/F26-Team-6/blob/main/Reports/Project_Proposal_Specifications.md [Accessed: Oct. 22 2026].


<br>

## Statement of Contributors



