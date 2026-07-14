# Hardware-Aware AI Proctoring Engine

## 1. Executive Summary
A production-grade, defense-in-depth security pipeline designed to maintain examination integrity. The system leverages a dual-subsystem architecture—combining lightweight CPU-based spatial geometry with asynchronous GPU-accelerated deep learning—to detect academic dishonesty in real-time without causing system thermal throttling on edge devices.

## 2. System Architecture
This project is built on a modular architecture designed to optimize compute loads across hardware components:

* **Phase 1: CPU-Based Liveness (The Sensor)** * Uses MediaPipe Face Mesh for landmark extraction.
  * Implements an Eye Aspect Ratio (EAR) state machine with a mathematical decay buffer to filter natural blinking from sustained visual anomalies.
* **Phase 2: Spatial Geometry (The Math)**
  * Utilizes `cv2.solvePnP` to map 2D landmarks to a 3D coordinate system using a digital intrinsic camera matrix.
  * Decomposes the rotation matrix to extract precise **Pitch** and **Yaw** Euler angles via linear algebra, tracking off-screen head movement natively on the CPU.
* **Phase 3: Asynchronous Object Detection (The Intelligence)**
  * Deploys **YOLOv8 Nano** via PyTorch, routed specifically to Nvidia CUDA cores.
  * Features a custom **15-frame skip logic** engine to optimize inference cycles, maintaining a 30 FPS geometry tracking feed while scanning for mobile devices twice per second.

## 3. Threat Model Matrix
The system flags anomalies based on a graduated risk-weighting strategy, utilizing a dynamic "Safe Box" deadzone to prevent math wrap-around vulnerabilities.

| Anomaly Flag | Detection Logic | Penalty Severity | Threat Intent |
| :--- | :--- | :--- | :--- |
| **Unauthorized Objects** | YOLOv8 (Class 67/73) | +10 | Direct access to outside information. |
| **Multiple Faces** | Face Mesh Tensor | +50 | Collaborative cheating (proxy test-taker). |
| **Head Pose Anomaly** | 3D Matrix Math (Pitch/Yaw) | +20 | Use of secondary monitor or desk notes. |
| **No Face Detected** | Face Tracking Loss | +25 | User left the testing perimeter. |
| **Eyes Closed** | EAR State Machine | +10 | Sustained inattention or listening to audio. |

## 4. Key Engineering Challenges Solved
* **Hardware Bottleneck:** Solved CPU/GPU load balancing by offloading spatial math to the CPU and heavy inference to the GPU.
* **Temporal Redundancy:** Implemented skip-frame inference to ensure the system remains performant on standard hardware, preventing UI lag.
* **Sensor Jitter:** Eliminated false positives through custom decay logic, ensuring that momentary natural behaviors or hardware flicker do not trigger security warnings.
* **The Kill Switch:** Integrated a deterministic 100-point risk threshold that terminates the process and locks the UI upon confirmed security breaches.

## 5. Technical Stack
* **Language:** Python
* **Computer Vision:** OpenCV, MediaPipe
* **Machine Learning:** PyTorch, Ultralytics YOLOv8
* **Hardware Optimization:** CUDA, NumPy, Matrix Decomposition
