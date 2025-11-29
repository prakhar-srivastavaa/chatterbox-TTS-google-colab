# ChatterBox TTS on Google Colab (T4 GPU)

Launch the [Chatterbox-TTS-Server](https://github.com/devnen/Chatterbox-TTS-Server) instantly on Google Colab with hardware acceleration. This repository contains a ready-to-run notebook plus sample audio so you can demo text-to-speech directly from your browser.

---

## 🚀 Quick Start

### 1. Spin It Up in Colab

Hit the badge to open the notebook with Colab’s GPU runtime already available:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/prakhar-srivastavaa/chatterbox-TTS-google-colab/blob/master/_Audio_ChatterBox_TTS.ipynb)

### 2. Switch the Runtime to GPU (Prefer T4)

- In Colab choose `Runtime → Change runtime type`.
- Set **Hardware accelerator = GPU**. T4 delivers the best balance of cost and throughput for this workflow.
- Use `Runtime → Managed session → View runtime info` to verify the T4 allocation.

### 3. Install Everything and Pull the Server

You can paste the cells below into a blank notebook, but the included `_Audio_ChatterBox_TTS.ipynb` already ships with these steps.

```python
# Fetch Chatterbox-TTS-Server
!git clone https://github.com/devnen/Chatterbox-TTS-Server.git
%cd Chatterbox-TTS-Server

# Torch stack built for CUDA 12.1 (matches Colab's T4 image)
!pip install torch==2.5.1+cu121 torchaudio==2.5.1+cu121 torchvision==0.20.1+cu121 \
	--index-url https://download.pytorch.org/whl/cu121 -q

# Core TTS components
!pip install git+https://github.com/devnen/chatterbox.git -q

# API server + audio helpers
!pip install fastapi uvicorn pyyaml soundfile librosa safetensors -q
!pip install python-multipart requests jinja2 watchdog aiofiles unidecode inflect tqdm -q
!pip install pydub audiotsm -q
```

### 4. Boot the FastAPI Server Window

```python
import gc, threading, time, uvicorn
from google.colab.output import serve_kernel_port_as_window

# Make sure port 8004 is free
!fuser -k 8004/tcp 2>/dev/null || echo "Port already free"
!pkill -f "uvicorn.*server" 2>/dev/null || echo "No server to kill"
gc.collect()

def launch_app():
		from server import app
		uvicorn.run(app, host="0.0.0.0", port=8004, log_level="warning", access_log=False)

print("🚀 Bringing up Chatterbox...")
thread = threading.Thread(target=launch_app, daemon=True)
thread.start()
time.sleep(25)  # allow model weights to load

print("🎉 Ready! Tap below to open the UI")
serve_kernel_port_as_window(8004)
```

---

## 📕 Repository Contents

- `_Audio_ChatterBox_TTS.ipynb` – Colab-friendly walkthrough with the entire setup.
- `result/` – directory where generated `.wav` files land automatically.
- `prakhar.wav`, `thermo.wav` – reference clips for quick validation.

---

## 📺 Video Companion

This codebase supports a walkthrough video that demonstrates:
- Installing dependencies end-to-end inside Colab.
- Reserving and verifying a T4 GPU runtime for faster inference.
- Launching the FastAPI server and opening its UI in a Colab window.

---

## 🔗 Useful Links

- [Chatterbox-TTS-Server](https://github.com/devnen/Chatterbox-TTS-Server)
- [Chatterbox core library](https://github.com/devnen/chatterbox)

---

## ⚡ Tips & Caveats

- First boot typically takes 20–30 seconds because the voice models must load into GPU memory.
- If Colab stops or restarts, simply re-run all cells; the server is stateless between sessions.
- This setup targets demos and experiments. For production deployments, containerize the server and add persistent storage/logging.

Contributions that improve the notebook UX, add new voices, or streamline audio export workflows are always welcome.
