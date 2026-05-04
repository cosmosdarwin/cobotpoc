# Simple Collaborative Robot

## Overview

This project combines the MyCobot 280 M5 robot arm with a microphone and a camera to achieve a simple, inexpensive "cobot" (collaborative robot) proof of concept. You can use voice to ask the robot to pick and place various objects on the table. The arm, microphone, and camera all connect using USB to the Linux computer running this project. See prerequisites for details.

## Features

The project is just a Python script that invokes these AI models:

- Speech (transcription): Nvidia Parakeet TDT 0.6b v2
- Language, reasoning, and tool calling: Qwen 2.5 1.5B Instruct
- Vision: Google's Hand Landmarker (hands), OWL-ViT Base Patch32 (other)

The robot arm executes coordinate-based movements (e.g., go to x, y, z) based on what it hears and sees.

## Prerequisites

You need:

- Computer with:
  - Nvidia GPU (developed using RTX 3050 8 GB)
  - Ubuntu (developed using 24.04 LTS)
- Peripherals:
  - MyCobot 280 M5 ($649)
  - UAC-compliant USB microphone
  - UVC-compliant USB camera

## Quick start

### 1. Install system-level dependencies:

- PortAudio (used by Python SoundDevice library)
- FFMPEG (used by Python OpenCV library)
- Python incl PIP, Virtual Environments, and python-is-python3 (for your sanity)

```bash
sudo apt update
sudo apt upgrade
sudo apt install usbutils
sudo apt install portaudio19-dev
sudo apt install ffmpeg
sudo apt install python3-dev python3-pip python3.12-venv python-is-python3

```

### 2. Nvidia driver:

Check if you have the full (closed-source, not-installed-by-default) Nvidia driver working:

```bash
nvidia-smi
```

If not, install the best driver version (depends on your hardware) and reboot:

```bash
sudo apt install nvidia-driver-580
sudo reboot
```

### 3. Create a Python virtual environment:

Necessary on Ubuntu to work around guardrails that discourage messing with system Python:

```bash
python -m venv cobotproject
cd cobotproject
source bin/activate
```

### 4. Install Python packages:

These will be added/modified only within your Python virtual environment:

```bash
pip install numpy scipy
pip install sounddevice soundfile
pip install opencv-python transformers torch
pip install nemo-toolkit[asr]
pip install pymycobot pyserial
pip install rich
```

> [!NOTE]
> Transformers is the HuggingFace library that lets you access their model catalog, download and cache weights, and do inferencing all within your Python code. Transformers will subsequently download the individual models you request on first use, like OWL-ViT (by Google) and Qwen (by Alibaba).

> [!NOTE]
> Installing Nvidia NeMo will pull in 2 GB+ of dependencies: CUDA, cuDNN, cuBLAS, and so on.

### 5. Download Hand_Landmarker model:

Unlike the other models which download at runtime from HuggingFace and Nvidia, the Hand_Landmarker model loads from file, so you need to download it (7.8 MB) yourself. In the project folder, run:

```bash
wget -q -O ./hand_landmarker.task https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task
```

### 6. Prepare MyCobot serial connection:

- Find the device path under `/dev/`, something like `ttyACM0`. This may change after each reboot.
- To instantiate robot in Python, set PORT = above, and BAUD = `115200` (fixed bit rate).
- Give yourself permission to the `dialout` group for accessing serial devices:

```bash
sudo usermod -aG dialout $USER
```

## How to run

```bash
python cobotpoc.py
```

Enjoy!