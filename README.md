# Real-Time 3D Dashcam Perception System

A computer vision-based dashcam perception system that transforms ordinary dashcam footage into a dual-panel visual perception interface, combining vehicle detection, depth estimation, object tracking, distance estimation, collision-risk classification, and 3D Bird's Eye View (BEV) visualization.

**Developer: Pragati Sarang**  
[GitHub](https://github.com/pragatisarang36-design)

## Project Overview

The application processes an input video frame by frame and creates a composed video with the resized dashcam view alongside a depth-colored, BEV-style visualization. It uses YOLOv8n to detect selected vehicle classes, Depth Anything V2 Small for monocular depth estimation, and Supervision's ByteTrack tracker to associate detections between frames.

For each tracked detection, the application keeps a short image-space trajectory and estimates image-space velocity from the change in bounding-box center between processed frames. It combines normalized depth values with a bounding-box projection heuristic to produce an approximate distance estimate, then assigns a color-coded collision-risk label. The second panel projects depth and vehicle detections into a 3D-style BEV visualization. These are visual estimates and should not be interpreted as calibrated real-world measurements.

The dashboard includes vehicle boxes and labels, trajectory traces, a lane-region overlay, depth visualization, and a HUD with processing FPS, frame progress, and selected compute device. Processing is performed on video frames; `--show` optionally opens a live preview window.

## Key Features

### Vehicle detection

- Uses the included YOLOv8n weights (`yolov8n.pt`).
- Detects cars, motorcycles, buses, trucks, and bicycles.
- Supports a configurable confidence threshold.

### Depth estimation

- Uses Depth Anything V2 Small through Hugging Face Transformers.
- Produces a per-frame normalized depth map and resizes it to the working frame dimensions.
- Applies temporal smoothing between consecutive depth maps.

### Object tracking

- Uses ByteTrack through Supervision to maintain tracking IDs.
- Stores recent object-center trajectories and estimates image-space velocity from frame-to-frame center changes.
- Draws trajectories and a direction arrow in the dashcam panel.

### Distance and collision-risk estimates

- Combines median depth within the detected bounding box with a bounding-box-width projection heuristic.
- Displays the resulting approximate distance beside each detection.
- Classifies detections as **SAFE**, **CAUTION**, **WARNING**, or **DANGER** using the estimated distance and horizontal position in the frame.
- Uses matching risk colors for detection boxes and BEV vehicle visualizations.

### BEV dashboard and monitoring

- Places the processed dashcam view and BEV-style visualization side by side.
- Colors a depth-based scene projection and overlays detected vehicles with 3D-style bounding boxes.
- Draws a lane-region overlay on the dashcam panel.
- Shows FPS, frame number and progress, and CPU or CUDA device information.

## Technologies Used

| Technology | Use |
| --- | --- |
| Python | Application and video-processing pipeline |
| OpenCV | Video input/output, image processing, and drawing |
| NumPy | Depth maps, projections, and array operations |
| PyTorch and Torchvision | Model runtime ecosystem |
| YOLOv8 (Ultralytics) | Vehicle detection |
| Depth Anything V2 Small | Monocular depth estimation |
| Hugging Face Transformers | Depth model pipeline |
| Supervision and ByteTrack | Detection conversion and multi-object tracking |
| tqdm | Frame-processing progress bar |
| FFmpeg | Optional output compression to H.264 MP4 |

## Project Structure

```text
Real-Time-3D-Dashcam-Perception-System/
├── app.py
├── yolov8n.pt
├── README.md
└── LICENSE
```

## Installation and Setup

The instructions below are for Windows and use Python 3.11, matching the existing project setup guidance. Run commands from PowerShell or Command Prompt.

### Clone the repository

```powershell
git clone https://github.com/pragatisarang36-design/Real-Time-3D-Dashcam-Perception-System.git
cd Real-Time-3D-Dashcam-Perception-System
```

### Create and activate a virtual environment

```powershell
python -m venv .venv
.venv\Scripts\activate
```

### Install dependencies

The following commands retain the project's pinned NumPy, PyTorch, Torchvision, SciPy, and Transformers versions and install the supporting packages:

```powershell
python -m pip install --upgrade pip
python -m pip install numpy==1.24.4
python -m pip install torch==2.2.0+cpu torchvision==0.17.0+cpu --index-url https://download.pytorch.org/whl/cpu
python -m pip install Pillow tqdm scipy==1.11.4 transformers==4.40.2 ultralytics supervision ffmpeg-python opencv-python matplotlib --no-deps
```

The script checks for missing Python dependencies when it starts and attempts to install them automatically. It also checks for the FFmpeg executable and attempts installation on Windows using `winget`, with a download fallback. If automatic FFmpeg installation does not work, install FFmpeg separately and make sure `ffmpeg` is available on `PATH`. The script can save the output without compression if FFmpeg is unavailable.

On a CUDA-capable system, install a PyTorch and Torchvision build compatible with your CUDA setup if needed. The automatic installer in this script installs the CPU builds when it detects missing or incompatible PyTorch packages.

## Running the Application

Pass the input video path as the positional argument:

```powershell
python app.py video.mp4
```

The processed video is saved to `output/final_output.mp4`.

### Command-line arguments

| Argument | Description | Default |
| --- | --- | --- |
| `input` | Path to the input dashcam video | Required |
| `--conf` | YOLO detection confidence threshold | `0.35` |
| `--device` | Compute device: `cpu` or `cuda` | `cpu` |
| `--show` | Open a live preview window; press `q` to stop preview processing | Disabled |

Examples:

```powershell
python app.py video.mp4 --show
python app.py video.mp4 --device cuda --show
python app.py video.mp4 --conf 0.5
```

The CUDA option uses CUDA only when PyTorch reports it is available; otherwise processing falls back to CPU.

## Disclaimer

This project is intended for educational use and computer vision research. Its distance estimates and collision-risk classifications are approximate visual estimates based on monocular depth and image geometry; they are not physically calibrated measurements or safety guarantees. This is not a certified Advanced Driver Assistance Systems (ADAS) product. Do not use it as a replacement for professional vehicle safety systems or for autonomous driving decision-making.
