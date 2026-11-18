# Valinor Dispatch — Condensed Research and Priority Design Choices

**Working draft | Updated November 18, 2026**  
**Confirmed processing hardware: NVIDIA Jetson Orin NX 16GB**

## 1. Main recommendations

| Category | Recommended choice | Reason for choosing it | Status |
|---|---|---|---|
| Processing hardware | **NVIDIA Jetson Orin NX 16GB** | The client confirmed this module. It has an 8-core Arm CPU, an NVIDIA Ampere GPU with 1,024 CUDA cores and 32 Tensor Cores, and 16GB of shared LPDDR5 memory. It provides significantly more headroom than the original Jetson Nano for video processing and AI inference. [1] | **Client-confirmed hardware** |
| AI/video pipeline | **NVIDIA DeepStream + GStreamer, matched to the installed JetPack release** | DeepStream combines video input, hardware-accelerated inference, buffering, and detection metadata. **DeepStream 7.1** is a documented option for JetPack 6; **DeepStream 9.1/9.2** corresponds to compatible JetPack 7 releases. Do not assume that the newest release or an older Nano-specific release matches the client's installation. [2][6] | Team decision; exact versions pending inspection |
| Object detector | **YOLOv4, using AlexeyAB/Darknet for the model workflow** | Detects visible drone components and outputs bounding boxes and confidence scores. The team can train or adapt the model, then investigate deployment through DeepStream/TensorRT instead of relying on the Darknet runtime on the Jetson. [3][4][16] | Team decision |
| Backup detector configuration | **YOLOv4-tiny, only if benchmarks justify it** | The Orin NX has more resources than the Nano, so full YOLOv4 should be tested first. The tiny model is a fallback if latency, thermal throttling, or memory usage is too high. [3][5] | Backup choice |
| Deployment/optimization | **TensorRT through DeepStream; benchmark FP16 first** | NVIDIA's accelerated inference stack can reduce inference latency. FP16 is a practical initial precision; INT8 may be explored after measuring accuracy and confirming supported calibration tools. [2][6] | Recommendation |
| Camera calibration/image checks | **OpenCV** | Corrects lens distortion, checks blur and brightness, and helps define fixed inspection regions. [11] | Recommendation |
| Annotation | **CVAT** | Provides image labeling and YOLO-format export for a drone-specific dataset. [12] | Recommendation |
| Training data | **Mostly project-collected dock images**, including controlled damage examples | The drone's position, lighting, camera angle, and background are specific to this dock. MVTec AD and LOCO AD are helpful references but do not replace labeled project images. [13][14] | Recommendation |
| RGB + thermal combination | **Late/decision-level fusion** | Process RGB and thermal information separately and combine their findings per component. This avoids requiring pixel-level alignment between a higher-resolution RGB camera and lower-resolution thermal sensor. | Recommendation based on team research |
| Reporting/storage | **Local JSON results, saved failure examples, and versioned logs** | Provides PASS/WARN/FAIL/RETRY results, component-level reasons, confidence, timestamps, and traceable software/model versions. | Recommendation |
| Hardware integration | **Check carrier-board interfaces, power, cooling, and storage** | The Orin NX is a module installed on a carrier board. The actual USB ports, power supply, cooling setup, and storage depend on that carrier and dock. Sustained performance needs testing under realistic operating conditions. [1] | New integration priority |

**Hardware/software compatibility note.** The team has confirmed the **Jetson Orin NX 16GB**, not the original Jetson Nano or Jetson Orin Nano. The previous recommendation of **DeepStream 6.0.1 + JetPack 4.6.1** was specific to the original Nano and should be removed from the deployment plan. NVIDIA lists **DeepStream 7.1** for the JetPack 6 family (including JetPack 6.2.3), **DeepStream 9.1** for JetPack 7.2, and **DeepStream 9.2** for JetPack 7.2.1. These are compatibility examples, not instructions to upgrade an already working client system. First record the exact JetPack/L4T, CUDA, TensorRT, DeepStream, and Ubuntu versions installed on the dock. [2][6]

**YOLOv4 integration note.** AlexeyAB/Darknet is the chosen YOLOv4 model implementation, but the deployed inference engine does not have to be Darknet itself. The third-party DeepStream-Yolo project documents Darknet-model conversion and DeepStream support through 8.0. A compatible integration and postprocessing parser must be tested for the actual DeepStream version; compatibility with 9.x should **not** be assumed from the repository alone. [16]

**YOLOv4 licensing note.** The AlexeyAB/Darknet repository uses a permissive public-domain-style license. This does **not** automatically determine the licenses for separately obtained pretrained weights, datasets, NVIDIA software, or other libraries. Those should be verified independently. [4]

## 2. Explanation of choices

- **DeepStream over OpenCV alone:** OpenCV remains useful for calibration and image-quality tests, but DeepStream offers an NVIDIA-optimized framework for camera streams, GPU inference, and detection metadata. The Jetson Orin NX supports more recent software stacks than the original Nano, provided the versions match the installed JetPack. [2]
- **YOLOv4 over a heavy two-stage detector:** YOLOv4 is the team's existing choice, so it makes sense to benchmark that before switching architectures. The Orin NX 16GB provides more headroom to test full YOLOv4. YOLOv4-tiny remains a fallback if the measured inspection time is unacceptable. [3][5]
- **TensorRT FP16 first:** The team should measure accuracy, time per image, end-to-end inspection time, CPU/GPU usage, and memory consumption using the actual camera input. The Orin NX's advertised peak AI performance is not a guarantee of YOLOv4 throughput on this project. [1][6]
- **Train on real dock images over online datasets alone:** Lighting, camera placement, drone model, and likely defect types are specific to this setup. The team needs images of normal components and safe, controlled examples of defects or missing parts.
- **Decision-level fusion over early or middle fusion:** The thermal sensor has much lower spatial resolution than the RGB camera. Independent processing reduces the need for exact pixel alignment and makes results easier to understand, while retaining the option to combine warnings by component.
- **Rules plus a detector over one giant AI model:** Simple component-presence checks can be handled by rules based on detections. More difficult damage or anomaly recognition can be added as a separate model if data and time permit.
- **Keep performance and hardware limits in view:** The Orin NX's 16GB is **shared system/GPU memory**, not dedicated GPU VRAM. Continuous inference should also be tested for power draw, operating temperature, and thermal throttling. Camera and storage bandwidth may affect the overall inspection time. [1]

## 3. Proposed system overview

**Dock starts inspection → RGB/thermal cameras and optional sensors capture data → image-quality checks → YOLOv4 locates drone components → RGB defect checks and thermal measurements → component-level decision fusion → PASS, WARN, FAIL, or RETRY → local report saved and sent to the dock controller.**

The **Jetson Orin NX 16GB** would run the main image-processing and AI inference tasks locally. The precise connection to the dock controller, camera ports, power supply, and storage will depend on the carrier board and the interface agreed upon with Valinor.

The audio check remains optional until the team and client decide what sound the system is meant to diagnose. A microphone cannot reliably detect a damaged propeller from a stationary motor. The system should not spin propellers automatically unless a separate test procedure and safety controls have been designed and approved.

## 4. First things to test

1. **Confirm the installed software and carrier board.** Record the Jetson Orin NX 16GB module, carrier-board model, JetPack/L4T, Ubuntu, CUDA, TensorRT, and DeepStream versions. Check available USB ports, storage, power supply, and cooling. Do not reflash the client's system without approval.
2. **Run a compatible DeepStream sample pipeline.** Use a DeepStream version documented for the existing JetPack installation and verify that the selected camera can supply frames without interrupting the dock software.
3. **Benchmark full YOLOv4 using TensorRT FP16.** Measure inference time, full inspection time, CPU/GPU utilization, shared memory consumption, and device temperature. Try YOLOv4-tiny only if full YOLOv4 cannot meet the agreed performance target.
4. **Validate YOLOv4 model conversion and output parsing.** Confirm that the chosen Darknet configuration/weights can be exported or integrated into the installed DeepStream/TensorRT stack. Recheck third-party compatibility if the Jetson runs DeepStream 9.x. [16]
5. **Check the RGB camera's inspection detail.** Verify that the entire area is visible and the smallest important defect can be resolved at the planned camera position.
6. **Check thermal capture and interpretation.** Confirm that the FLIR Lepton/PureThermal combination produces usable radiometric or temperature-related information, rather than only a false-color display. Identify what thermal measurements the client actually needs.
7. **Collect repeatable dock images.** Capture healthy components from realistic dock positions. Start with a component checklist such as propellers, motors, arms, body, battery, and visible fasteners; add controlled defect examples when safe.
8. **Agree on the interface and decision rules.** Define how the dock starts an inspection, receives JSON results, handles timeouts, and distinguishes PASS, WARN, FAIL, and RETRY. Set measurable accuracy and inspection-time targets with Valinor.

## 5. References

[1] NVIDIA, [Jetson Orin module specifications](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/) and [Jetson Orin NX 16GB overview](https://developer.nvidia.com/blog/boost-edge-ai-performance-with-the-new-nvidia-jetson-orin-nx-16gb/).  
[2] NVIDIA, [DeepStream installation and platform/OS compatibility matrix](https://docs.nvidia.com/deepstream/text/DS_Installation).  
[3] A. Bochkovskiy et al., [YOLOv4: Optimal Speed and Accuracy of Object Detection](https://arxiv.org/abs/2004.10934), 2020.  
[4] AlexeyAB, [Darknet YOLOv4 repository and license](https://github.com/AlexeyAB/darknet/blob/master/LICENSE).  
[5] C.-Y. Wang et al., [Scaled-YOLOv4](https://arxiv.org/abs/2011.08036), 2020.  
[6] NVIDIA, [JetPack SDK 6.2.3](https://developer.nvidia.com/embedded/jetpack-sdk-623) and [DeepStream SDK support matrix](https://catalog.ngc.nvidia.com/orgs/nvidia/-/collections/deepstream_sdk).  
[7] Arducam, [B0441 5 MP UVC camera](https://www.arducam.com/5mp-autofocus-usb-camera-module-with-single-microphone-for-windows-linux-macos-android.html).  
[8] GroupGets, [FLIR Lepton 3.5 listing](https://groupgets.com/collections/thermal-imaging).  
[9] GroupGets, [PureThermal 2 Shielded](https://groupgets.com/products/purethermal-2-shielded).  
[10] Samson, [Go Mic specifications](https://www.samsontech.com/products/microphones/usb-microphones/gomic/).  
[11] OpenCV, [Camera calibration reference](https://docs.opencv.org/4.x/d9/d0c/group__calib3d.html).  
[12] CVAT, [YOLO annotation format](https://docs.cvat.ai/docs/dataset_management/formats/format-yolo/).  
[13] MVTec, [MVTec AD](https://www.mvtec.com/research-teaching/datasets/mvtec-ad).  
[14] MVTec, [MVTec LOCO AD](https://www.mvtec.com/research-teaching/datasets/mvtec-loco-ad).  
[15] M. Batzner et al., [EfficientAD: Accurate Visual Anomaly Detection at Millisecond-Level Latencies](https://openaccess.thecvf.com/content/WACV2024/html/Batzner_EfficientAD_Accurate_Visual_Anomaly_Detection_at_Millisecond-Level_Latencies_WACV_2024_paper.html), WACV 2024.  
[16] marcoslucianops, [DeepStream-Yolo compatibility and configuration examples](https://github.com/marcoslucianops/DeepStream-Yolo) (third-party; pin and test a compatible revision, especially for newer DeepStream versions).

**Source notes:** This shortlist was developed from the team's *AI Pipelining Integration Research*, the prior condensed research draft, the client's updated processing-board information, and the references above.
