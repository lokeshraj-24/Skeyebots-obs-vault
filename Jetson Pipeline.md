
Use the SIYI gimbal Ethernet→RJ45 cable
# Verify it worked

1. **Link light**: LEDs on the RJ45 ports should be on/blinking.
    
2. **Check address**
    
    `ip addr show enp3s0 # look for 192.168.144.20/24`
    
3. **Ping the camera**
    
    `ping 192.168.144.25`
    
4. **Probe RTSP quickly (optional)**
    
    `ffprobe -rtsp_transport tcp rtsp://192.168.144.25:8554/main.264`
    
5. **Play test (no code)**
    
    `gst-launch-1.0 -e \   rtspsrc location=rtsp://192.168.144.25:8554/main.264 latency=80 protocols=tcp ! \   rtph264depay ! h264parse ! avdec_h264 ! videoconvert ! autovideosink`


Jetson to Camera setup:


Yep—let’s get you from “script on laptop” ➜ “running on Jetson (JP 5.1.4) with GStreamer + Ultralytics” cleanly.

Because your earlier `pip install` hit a weird package (`puccinialin`), I strongly recommend **starting a fresh venv** on the Jetson to avoid inherited junk. If you really want to reuse the old venv you can, but a clean one saves time.

```

# B) Prep Jetson system packages (OpenCV+GStreamer from apt)

_SSH into Jetson:_

```bash
ssh your_jetson_user@JETSON_IP
sudo apt update
sudo apt install -y python3-opencv \
  gstreamer1.0-tools gstreamer1.0-plugins-{base,good,bad} gstreamer1.0-libav \
  python3-venv python3-pip git build-essential libjpeg-dev zlib1g-dev python3-dev
```

> Keep **apt OpenCV** (has GStreamer/NVDEC). **Do not** `pip install opencv-python` on Jetson.

# C) Make a **fresh** venv that can see system OpenCV

```bash
cd ~/projects/a8_listener
python3 -m venv .venv --system-site-packages
source .venv/bin/activate
python -m pip install -U pip setuptools wheel
pip cache purge
```

_If you prefer to reuse the old venv:_

```bash
source ~/projects/a8_listener/.venv/bin/activate
python -m pip install -U pip setuptools wheel
pip cache purge
```

# E) Install Ultralytics (without pulling conflicting deps)

```bash
pip install --no-deps ultralytics
pip install numpy pillow pyyaml tqdm scipy psutil matplotlib
```



## F) Sanity checks (OpenCV has GStreamer; torch is OK)

```bash
python - << 'PY'
import cv2, torch, torchvision, ultralytics
print("cv2 from:", cv2.__file__)
print("GStreamer enabled in OpenCV?", "GStreamer" in cv2.getBuildInformation())
print("torch:", torch.__version__, "cuda?", torch.cuda.is_available())
print("torchvision:", torchvision.__version__)
print("ultralytics:", ultralytics.__version__)
PY
```

    
---

If anything errors, paste the **last ~60 lines** of the output. I’ll spot the missing piece (usually a small dev lib or a mismatch in the codec pipeline).





FINAL gft CONFIG of A82Jetson

gst = (
    f"rtspsrc location={RTSP} latency=80 protocols=tcp drop-on-latency=true ! "
    "rtph265depay ! "
    "h265parse config-interval=1 disable-passthrough=true ! "
    "video/x-h265,stream-format=byte-stream,alignment=au !"
    "nvv4l2decoder ! "
    "nvvidconv ! video/x-raw,format=BGRx ! "
    "videoconvert ! video/x-raw,format=BGR ! "
    "appsink drop=true sync=false max-buffers=1"
)

cap = cv2.VideoCapture(gst, cv2.CAP_GSTREAMER)

Output type:
results = model(frame, verbose=False)
    annotated = results[0].plot()




JETSON TO HM30:

Before connecting a third-party Ethernet camera to SIYI link, please change its IP address to “192.168.144.X”. Mark The “X” should not be “11”, “12”, or “20”, otherwise it won’t work. The three addresses have been occupied by the ground unit, the air unit, and Android system of SIYI handheld ground station.

Interested classes: 
Humans
Vehicles
Fire
Smoke





gst_out = (
    f'appsrc is-live=true format=time do-timestamp=true '
    f'caps=video/x-raw,format=BGR,width={W},height={H},framerate={FPS}/1 ! '
    'videoconvert ! video/x-raw,format=NV12 ! '
    'nvv4l2h264enc preset-level=1 insert-sps-pps=true iframeinterval=30 bitrate=4000000 ! '
    'h264parse ! rtph264pay pt=96 config-interval=1 ! '
    f'udpsink host={DEST_IP} port={DEST_PORT} sync=false'
)



FFPLAY RSPCLIENT TO PULL FROM JETSON:


ffplay rtsp://192.168.144.112:8554/stream -rtsp_transport tcp

numpy-old version: 1.17


## Setup Jetson env for pipeline:

Download opencv from sudo apt
```
sudo apt install -y python3-opencv   gstreamer1.0-tools gstreamer1.0-plugins-{base,good,bad} gstreamer1.0-libav   python3-venv python3-pip git build-essential libjpeg-dev zlib1g-dev python3-dev
```

If new env, do:
```
python3 -m venv pipeline --system-site-packages
```

if pipeline env already created:
```
python3 -m venv --system-site-packages pipeline
```

Just in case(not 100% is its mandatory):
```
python -m pip install -U pip setuptools wheel
```

Installing torch, torchvision:

```
pip install "numpy==1.21"
```

```
sudo apt-get install -y libjpeg-dev zlib1g-dev libpng-dev libopenblas0 libopenblas-dev libgfortran5 ninja-build
```

```
git clone --branch v0.16.2 https://github.com/pytorch/vision.git
cd vision
export FORCE_CUDA=1
export USE_FFMPEG=0
export USE_OPENCV=0          # <-- keeps torchvision from linking to OpenCV
export BUILD_VERSION=0.16.2
export TORCH_CUDA_ARCH_LIST="8.7" 
python setup.py bdist_wheel
python -m pip install --no-deps --no-build-isolation -v .

```

```
pip install --upgrade "packaging>=24.0" "setuptools>=69,<75" "wheel>=0.41" "setuptools-scm>=8"
sudo apt-get update
```


Multi-cast RTSP:

Client command:
```
ffplay -rtsp_transport udp_multicast rtsp://192.168.144.112:8554/stream
```


