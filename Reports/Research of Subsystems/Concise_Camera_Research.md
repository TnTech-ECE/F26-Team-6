# UVC Camera Integration

## General Overview
### Protocol Choice
Employing UVC cameras combines a standard physical USB interface with an OS-native class driver, eliminating the need for vendor-specific drivers while standardizing both the descriptor format and video streaming interface[cite: 1]. While the maximum theoretical bandwidth of USB is lower than that of dedicated MIPI CSI-2 lanes, the cost-to-performance ratio, component availability, and plug-and-play simplicity make UVC modules far more practical for deployment[cite: 1]. Furthermore, because our architecture captures high-resolution still images sequentially rather than streaming concurrent live video, raw bus bandwidth is never an operational bottleneck[cite: 1]. Altogether, UVC cameras provide a robust, modular vision pipeline that can easily be ported across host platforms (Linux, Windows, and macOS) without requiring custom kernel drivers or low-level re-engineering[cite: 1].

### Functionality and Interfacing
Depending on the enclosure layout, multiple cameras are required to fully inspect the drone's geometry and component plane[cite: 2]. Because embedded platforms such as the NVIDIA Jetson feature a limited number of native USB root ports, an externally powered USB hub is integrated into the dock architecture[cite: 1]. The potential risk of bus saturation across a shared hub is mitigated by the dock's deterministic, sequential inspection routine: the compute unit queries one camera at a time, pulls the maximum-resolution frame, flushes the buffer, and releases the endpoint before polling the next[cite: 1, 4].

On the Jetson's native Linux environment, UVC devices automatically register as standard Video4Linux2 (V4L2) endpoints (`/dev/video*`)[cite: 1]. This allows direct, low-overhead image ingestion into standard open-source computer vision pipelines such as OpenCV (`cv2.VideoCapture`)[cite: 1]. 

For machine learning defect detection, the project requires an architecture that is fully open-source, easily portable, and flexible for custom modifications:
* **YOLO Framework Trade-Offs:** The team evaluated open-source YOLO architectures, specifically considering **YOLOv4** (Darknet framework) for its permissive licensing and complete freedom to modify, optimize, and protect proprietary source modifications without commercial licensing encumbrances. In parallel, newer frameworks like **YOLOv8** offer turn-key defect-detection tools (such as `drone-defect-detector`)[cite: 2].
* **Unsupervised Anomaly Detection:** In addition to supervised bounding-box models, the vision pipeline leverages open-source unsupervised frameworks such as **Anomalib (PatchCore/PaDiM)** optimized via OpenVINO[cite: 1, 2]. This enables the dock to identify irregular surface cracks, chips, and blade delamination against a healthy baseline without requiring extensive defective training samples[cite: 1, 2].



## Physical Layout Plan
### Givens and Design Constraints
* **Drone Docking Station Enclosure:** $30"\text{ (W)} \times 40"\text{ (L)} \times 12"\text{/16" (H)}$
* **Drone Footprint:** $20"\text{ (W)} \times 20"\text{ (L)} \times 6"\text{/8" (H)}$, positioned precisely in the center of the docking station
* **Smallest Feature Size:** $0.5\text{ mm}$ (hairline cracks, surface chips, missing fasteners)
* **Target Pixel Density:** $3\text{ pixels per feature size} \rightarrow \mathbf{6\text{ pixels/mm}}$


### General Measurements and Geometric Equations
* **Distance from 40" wall to drone edge:** $5\text{ inches}$[cite: 1]
* **Distance from 30" wall to drone edge:** $10\text{ inches}$[cite: 1]
* **HFOV / VFOV (Equal Right-Angle Triangles):** 
  $$\text{FOV} = 2 \times \arctan\left(\frac{o}{a}\right)$$
* **HFOV / VFOV (Unequal Right-Angle Triangles):** 
  $$\text{FOV} = 90^\circ - \arctan\left(\frac{o_1}{a_1}\right) - \arctan\left(\frac{o_2}{a_2}\right)$$
* **Required Resolution / Field of View Equation:**
  $$\text{Physical FOV (mm)} = 2 \times \text{WD} \times \tan\left(\frac{\theta}{2}\right)$$
  *(where $\text{WD}$ is the maximum working distance needed to be resolved)*


### Model Evaluation & Optical Layout Analysis

#### Model 1: Midpoint Wall-Mounted Cameras
* **Configuration:** Cameras placed at the exact midpoint of the docking station walls (e.g., at the 20" mark on the 40" wall) at half the height of the drone.
* **The Math & Trade-Offs:**
  * Placing a camera on the 40" wall looking across the 30" span places the lens only $5\text{ inches}$ away from the drone edge[cite: 1]. 
  * Capturing the $20"$ width from this short distance requires an ultra-wide horizontal FOV of **126.86°**[cite: 1].
  * *Implication:* While UVC lenses exceeding 130° are readily available[cite: 1], they introduce heavy barrel (fisheye) distortion, necessitating rigorous OpenCV calibration de-warping[cite: 1].

#### Model 2: Corner-Mounted Cameras (Caddy-Corner Configuration)
* **Configuration:** Positioning the four visual UVC cameras directly in the extreme corners of the $30" \times 40"$ box, angled inward toward the center.
* **The Math & Trade-Offs:**
  * By moving the cameras into the corners, the working distance to the center of the box increases to roughly $25\text{ inches}$, and the target span becomes the $28.28"$ diagonal of the square drone[cite: 1].
  * The required horizontal field of view drops significantly to **under 90°** (specifically around **59°** to comfortably capture the diagonal span)[cite: 1].
  * *Implication:* This layout avoids extreme fisheye distortion, yields higher native pixel density on micro-defects[cite: 1], and provides a uniform perspective across all four symmetrical endpoints.

<br>

## Camera Recommendation
For the visual inspection subsystem, the architecture utilizes a **caddy-corner configuration**, mounting four high-resolution UVC cameras directly in the extreme corners of the 30" × 40" docking station enclosure, tilted downward and angled inward toward the center[cite: 1, 2, 4]. 

* **Optical Advantage & Distortion Minimization:** Positioning the cameras in the corners increases the working distance to the center to roughly $25\text{ inches}$, aligning the target span with the $28.28\text{"}$ diagonal of the centered 20" × 20" drone[cite: 1, 3]. This geometry reduces the required horizontal field of view (HFOV) to approximately **59°**, completely eliminating the severe barrel (fisheye) distortion associated with ultra-wide wall-mounted lenses while preserving maximum pixel density for micro-defect detection[cite: 1, 3].
* **Sensor Selection (12.3MP & Sony IMX500):** To achieve the required target density of 6 pixels per millimeter for 0.5 mm features, the subsystem specifies **12.3-megapixel camera modules**, with a notable mention of modules powered by the **Sony IMX500** sensor[cite: 1, 4]. These sensors deliver ultra-high-definition resolution, ensuring hairline cracks, surface chips, and missing fasteners are clearly resolved without requiring complex multi-camera stitching [cite: 1, 4].

<br>

## Software Compatibility
The software pipeline is containerized using Docker to ensure cross-platform portability and ease of deployment across edge hardware[cite: 4]:
* **Linux & YOLO Frameworks:** On Linux-based edge compute platforms (such as the NVIDIA Jetson Orin Nano Super), the architecture deploys **YOLO** (optimized via TensorRT/CUDA) for supervised object detection, reliably locating structural fasteners and hardware anomalies[cite: 1, 4]. Notably, YOLOv4 stands out as one of the best for the project since it allows for any modded code to be private if the client wants it to be.
 
* **Intel & OpenVINO Runtime:** For Intel-based x86 controllers or mini-PCs, execution is accelerated using the **OpenVINO toolkit**, optimizing inference latency and throughput for unsupervised anomaly detection libraries like **Anomalib** (PatchCore/PaDiM)[cite: 1, 2, 4]. 
(Note: This is only an option, if it is deemed necessary to swap devices due to computing power issues or anything of the sort.) 

* **Image Ingestion & Preprocessing:** Camera modules interface natively via Video4Linux2 (`/dev/video*`) endpoints using standard **OpenCV** (`cv2.VideoCapture`) pipelines, which execute initial distortion calibration matrices and Variance of the Laplacian sharpness quality gates prior to model inference[cite: 1, 3, 5].

<br>

## Connections
The peripheral array relies on a standardized, modular physical and protocol architecture:
* **UVC over USB Interface:** All visual and thermal imaging modules communicate via the standardized **USB Video Class (UVC)** protocol over physical USB connections, requiring zero custom kernel drivers or vendor SDK extensions across Linux, Windows, or macOS platforms[cite: 1, 5].
* **Powered USB Hub & Port Aggregation:** To accommodate the 4 caddy-corner visual cameras, the top-down thermal imager (FLIR Lepton UW / 3.5 via PureThermal), and the USB acoustic microphone within the physical limits of the edge compute board, an internal **externally powered USB hub** is integrated[cite: 1, 4].
* **Bandwidth Management via Sequential Polling:** To prevent USB root hub controller saturation from high-resolution 12.3MP concurrent streams, the system executes an event-driven **sequential snapshot capture loop**, querying one camera at a time, flushing ring buffers, and closing endpoints before proceeding[cite: 1, 4].

## Ethical, Professional, and Standards
| Standard or specification | Relevance to your subsystem | How your team could apply it |
|---|---|---|
| ISO 12233:2024 | Camera resolution and spatial-frequency response. | Test whether the camera and lens preserve enough detail to detect the project's minimum defect size. |
| ISO 15739:2023 | Digital-camera noise and dynamic-range measurements. | Evaluate image quality under the lighting conditions expected inside the docking station. |
| USB-IF UVC / USB Video specifications | Camera-to-compute interface compatibility. | Document supported video formats, resolutions, transfer modes, and host compatibility. Confirm the exact specification and version supported by each camera. |
| IEC 62368-1 | Electrical and fire safety of applicable audio/video and information-technology equipment. | Use as a safety reference when reviewing powered electronics, hubs, enclosures, and the overall equipment assembly. |


## References

[1] CAPSTONE Project Research & Subsystem Architecture Notes, 2026.

[2] Edge AI and Vision Alliance, "What is a UVC Camera, and What are the Different Types of UVC Cameras?," Edge AI and Vision Alliance, Jan. 2024. [Online]. Available: https://www.edge-ai-vision.com/2024/01/what-is-a-uvc-camera-and-what-are-the-different-types-of-uvc-cameras/. [Accessed: Sep. 28, 2026].

[3] Edge AI and Vision Alliance, "Unlock Edge AI Potential: Why USB Cameras Are the Future of Vision Solutions," Edge AI and Vision Alliance, Nov. 2023. [Online]. Available: https://www.edge-ai-vision.com/2023/11/unlock-edge-ai-potential-why-usb-cameras-are-the-future-of-vision-solutions/. [Accessed: Sep. 28, 2026].

[4] Edge AI and Vision Alliance, "Choosing Best USB Camera for Edge AI Retail Systems," Edge AI and Vision Alliance, Dec. 2023. [Online]. Available: https://www.edge-ai-vision.com/2023/12/choosing-best-usb-camera-for-edge-ai-retail-systems/. [Accessed: Sep. 28, 2026].

[5] e-con Systems, "USB 3.0 vs. MIPI CSI-2: Choosing the Right Interface for High-Resolution Embedded Vision Systems," e-con Systems, 2023. [Online]. Available: https://www.e-consystems.com/blog/camera/technology-explained/usb-3-0-vs-mipi-csi-2-choosing-the-right-interface-for-high-resolution-embedded-vision-systems/. [Accessed: Sep. 28, 2026].

[6] OpenCV, "Camera Calibration and 3D Reconstruction (calib3d)," OpenCV Documentation. [Online]. Available: https://docs.opencv.org/5.0/tutorials/calib3d/camera_calibration/camera_calibration.html. [Accessed: Sep. 28, 2026].

[7] Intel, "Optimize Inference — OpenVINO™ Documentation," Intel Corporation, 2024. [Online]. Available: https://docs.openvino.ai/2026/openvino-workflow/running-inference/optimize-inference.html. [Accessed: Sep. 28, 2026].

[8] Intel, "Anomalib: An Anomaly Detection Library for Deep Learning," GitHub repository, 2024. [Online]. Available: https://github.com/openvinotoolkit/anomalib. [Accessed: Sep. 28, 2026].

[9] biscolab, "drone-defect-detector: Automated defect detection pipeline using YOLOv8," GitHub repository, 2024. [Online]. Available: https://github.com/biscolab/drone-defect-detector. [Accessed: Sep. 28, 2026].

[10] H. Ramezani, "drone-defect-detection: Structural surface defect classification with synthetic data," GitHub repository, 2024. [Online]. Available: https://github.com/HoomanRamezani/drone-defect-detection. [Accessed: Sep. 28, 2026].

[11] Gemini (model: Flash-Lite), personal AI collaborator by Google, utilized for technical editing, formatting, grammar correction, and structural refinement of engineering documentation, 2026.
