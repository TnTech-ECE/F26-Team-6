# Valinor Dispatch — Condensed Research and Priority Design Choices

**working draft | October 17, 2026**  

## 1. Main recommendations

| Category | Recommended choice | Reason for choosing it | Status |
|---|---|---|---|
| AI/video pipeline | **NVIDIA DeepStream 6.0.1 + GStreamer** | Provides camera streaming, GPU inference, buffering, and detection metadata in one pipeline. The original Nano is supported by DeepStream 6.0.1 with JetPack 4.6.1. [2] | Team decision; version must be checked |
| Object detector | **YOLOv4 using the AlexeyAB/Darknet implementation** | Locates drone parts and reports bounding boxes and confidence scores. The Darknet repository's license is permissive, fitting the team's preference not to release its own source code. [3][4] | Team decision |
| Backup detector configuration | **YOLOv4-tiny, only if needed** | Much lighter than full YOLOv4, so it is worth testing if full YOLOv4 is too slow or uses too much Nano memory. [5] | Backup detector choice |
| Deployment | **TensorRT through DeepStream; test FP16 first** | Gives a reasonable way to accelerate the detector on the Jetson. Actual speed and accuracy must be measured. [2][6] | Recommendation |
| Camera calibration / image checks | **OpenCV** | Can correct distortion, check blur and brightness, and help set fixed inspection regions. [11] | Recommendation |
| Annotation | **CVAT** | Free/open tool for labeling images with boxes; can export YOLO labels. [12] | Recommendation |
| Training data | **Mostly project-collected dock images**, including controlled damage examples | A drone in this dock will look different from objects in public datasets. Public MVTec AD / LOCO AD data is useful for research, not a substitute for the actual drone dataset. [13][14] | Recommendation |
| RGB + thermal combination | **Late/decision-level fusion** | Keep RGB inspection and thermal measurements separate, then combine their results by component. Easier to debug than training one large combined network. | Recommendation based on team research |
| Reporting/storage | **Local JSON results + saved failure examples + versioned logs** | The dock needs a simple output with PASS/WARN/FAIL, reasons, confidence, and timestamps. | Recommendation |

The Nano must be confirmed to be the original Jetson Nano, not an Orin Nano. NVIDIA's compatibility information lists **DeepStream 6.0.1 / JetPack 4.6.1** for the original Nano; newer DeepStream examples may not run on it. The team should record the currently installed JetPack, DeepStream, CUDA, and TensorRT versions before making changes. [2][6]

**YOLOv4 licensing note.** The recommended AlexeyAB/Darknet repository has a permissive license. That does **not** automatically cover every external pretrained weight, custom data source, DeepStream component, or third-party library. Those licenses should be checked separately. [4]

## 2. Explanation of choices

- **DeepStream over OpenCV alone:** OpenCV is useful for image processing and quick tests, but DeepStream gives the project a better starting point for GPU-accelerated video processing on NVIDIA hardware. OpenCV will still be used for calibration and checks. The current team's pipeline research compared DeepStream, OpenCV, and GStreamer.
- **YOLOv4 over a heavy two-stage detector:** YOLOv4 is already the team's choice and is more practical for edge object detection than starting with Faster R-CNN. Full YOLOv4 must still be benchmarked on the Nano; YOLOv4-tiny is the backup. [3][5]
- **Train on real dock images over using only online datasets:** Lighting, background, camera angle, and the drone model are unusually specific. The team needs its own healthy and damaged images even if public datasets are used to learn the workflow.
- **Decision-level fusion over early or middle fusion:** The thermal camera has much lower resolution. Separate analysis avoids forcing both cameras into one pixel-aligned neural network and makes component-level warnings easier to explain.
- **Rules plus a detector over one giant AI model:** Rules are more dependable for simple checks such as whether a required part was detected. Small defect recognition can be added as a separate model if time and data permit.

## 3. Proposed system overview

**Dock starts inspection → lights and sensors capture data → quality checks → YOLOv4 locates drone components → RGB/thermal/audio checks → decision rules → PASS, WARN, FAIL, or RETRY → report saved and sent to dock.**

The audio check is optional until the team and client agree what sound the system is supposed to diagnose. A microphone by itself cannot find damaged propellers while the motors are stopped. The system should not automatically spin the propellers during inspection unless the team designs and approves a separate safe test.

## 4. First things to test

1. Identify the Jetson Nano board and installed JetPack/DeepStream software; check whether DeepStream 6.0.1 can be used without disrupting existing dock software. If not then buy a new jetson nano board to integrate into dock.
2. Run a sample DeepStream pipeline and a pretrained YOLOv4 model, and log RAM usage and inference time. Try YOLOv4-tiny only if needed.
3. Confirm that the chosen RGB camera can image the entire area and still show the smallest defect the client cares about.
4. Confirm that the thermal USB board exposes usable temperature-related data, not only a false-color video image.
5. Collect repeatable healthy images from actual dock positions. Create a simple component list (propellers, motors, arms, body, battery, and any visible fasteners).
6. Agree on the dock input/output contract and which failures should cause WARN, FAIL, or RETRY.

## 5. References

[1] NVIDIA, [Jetson Nano specifications](https://developer.nvidia.com/embedded/jetson-nano/).  
[2] NVIDIA, [DeepStream platform/OS compatibility matrix](https://docs.nvidia.com/metropolis/deepstream-nvaie31/dev-guide/text/DS_Quickstart.html).  
[3] A. Bochkovskiy et al., [YOLOv4: Optimal Speed and Accuracy of Object Detection](https://arxiv.org/abs/2004.10934), 2020.  
[4] AlexeyAB, [Darknet YOLOv4 repository and license](https://github.com/AlexeyAB/darknet/blob/master/LICENSE).  
[5] C.-Y. Wang et al., [Scaled-YOLOv4](https://arxiv.org/abs/2011.08036), 2020.  
[6] NVIDIA, [JetPack SDK 4.6.1](https://developer.nvidia.com/embedded/jetpack-sdk-461) and [DeepStream 6.0.1 release notes](https://docs.nvidia.com/metropolis/deepstream/DeepStream_6.0.1_Release_Notes.pdf).  
[7] Arducam, [B0441 5 MP UVC camera](https://www.arducam.com/5mp-autofocus-usb-camera-module-with-single-microphone-for-windows-linux-macos-android.html).  
[8] GroupGets, [FLIR Lepton 3.5 listing](https://groupgets.com/collections/thermal-imaging).  
[9] GroupGets, [PureThermal 2 Shielded](https://groupgets.com/products/purethermal-2-shielded).  
[10] Samson, [Go Mic specifications](https://www.samsontech.com/products/microphones/usb-microphones/gomic/).  
[11] OpenCV, [camera calibration reference](https://docs.opencv.org/4.x/d9/d0c/group__calib3d.html).  
[12] CVAT, [YOLO annotation format](https://docs.cvat.ai/docs/dataset_management/formats/format-yolo/).  
[13] MVTec, [MVTec AD](https://www.mvtec.com/research-teaching/datasets/mvtec-ad).  
[14] MVTec, [MVTec LOCO AD](https://www.mvtec.com/research-teaching/datasets/mvtec-loco-ad).  
[15] M. Batzner et al., [EfficientAD: Accurate Visual Anomaly Detection at Millisecond-Level Latencies](https://openaccess.thecvf.com/content/WACV2024/html/Batzner_EfficientAD_Accurate_Visual_Anomaly_Detection_at_Millisecond-Level_Latencies_WACV_2024_paper.html), WACV 2024.  
[16] marcoslucianops, [DeepStream-Yolo compatibility/configuration examples](https://github.com/marcoslucianops/DeepStream-Yolo) (third-party implementation; pin a compatible revision and test on Nano).  

**Source notes:** This shortlist was developed from the team's *AI Pipelining Integration Research*, along with the linked documentation above.
