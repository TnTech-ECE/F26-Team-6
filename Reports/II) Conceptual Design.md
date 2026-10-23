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

### Customer Specifications
- The chosen design shall
- The chosen design shall
- The chosen design shall

### Team 6's Ingenuity
- The chosen design shall
- The chosen design shall
- The chosen design shall

### Standards of Compliance
- The chosen design shall
- The chosen design shall
- The chosen design shall

### Ethical Concerns and Broader implication
- The chosen design shall
- The chosen design shall
- The chosen design shall

<br>

## Comparitive Analysis of Potential Solutions 
&nbsp;&nbsp;&nbsp; The project proposal previously described five main subsystems: machine vision via UVC cameras, thermal imaging, acoustics, lighting, and an AI/machine learning pipeline. For the following section, "Comparitive Analysis of Potential Solutions", it details the process of design constraints that revealed pros and cons of different solutions. Moreover, a comparative anaylsis of design methods and evaluation of COTs (Commercial off-the-shelf) devices were explored to meet the needs of Team 6's prototype.

**Team 6's prototype objectives:**

&nbsp;&nbsp;&nbsp; The project is intended to remain portable across future drone models and computing platforms. The first prototype may use a particular embedded computer, camera, or microphone, but the architecture should not require one vendor's product to remain functional. Specifically, each device or subsystem should be reliable, easily deployable, and designed to minimize inaccurate results. Altogether, an overengineered design should be presented for each system to achieve each objective. As for cost and budgeting, each choice should come with a detailed justification, as a total budget was not defined by Valinor Dispatch.

&nbsp;&nbsp;&nbsp; Each subsystem section—machine vision via UVC cameras, thermal imaging, acoustics, lighting, and an AI/machine learning pipeline—covers design, hardware recommendations, calculations, and physical layout where applicable. Content will vary by subsystem, with some requiring fewer topics and others including additional material.

### Machine Vision

### Thermal Imager

### Acoustics 

### Lighting 

### AI/Machine Learning 


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



