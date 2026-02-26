

## Environment setup in U24 with pytorch (GPU enabled), ultralytics, opencv with GStreamer support


```bash
sudo apt update
```

1) Install Gstreamer:
```bash
sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \ libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base \ gstreamer1.0-plugins-good gstreamer1.0-plugins-bad \ gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-tools \ gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3 \ gstreamer1.0-qt5 gstreamer1.0-pulseaudio
```

2) Install opencv from apt: 
```bash
sudo apt install python3-opencv libopencv-dev
```

3) Create conda env with python=3.12 (required for ubuntu 24):

```bash
conda create -n sk_cv python=3.12
conda activate sk_cv
```

4) Create the link between the conda env and the original opencv with Gstreamer in base env
```bash
cd $CONDA_PREFIX/lib/python3.12/site-packages/
ln -s /usr/lib/python3/dist-packages/cv2.cpython-312-x86_64-linux-gnu.so cv2.so
```

5) Install dependencies that are required to run opencv in conda
	- ubuntu 24's python is compiled against np<2.0, using np>2.0 will break opencv
```bash
pip install "numpy<2.0"
pip3 install --pre torch torchvision torchaudio
pip install ultralytics --no-deps
pip install requests pandas seaborn tqdm py-cpuinfo psutil scipy matplotlib pyyaml	  
```

**Terminal code to check opencv with Gstreamer support:**
```bash
python -c "
import cv2
print(cv2.getBuildInformation())
"
```




## Drone Jetson setup:

```bash
sudo apt update
```

##### Install opencv from debian
```bash
sudo apt install python3-opencv libopencv-dev
```

##### Install Gstreamer dependencies

```bash
sudo apt-get install -y \
    libgstreamer1.0-dev \
    libgstreamer-plugins-base1.0-dev \
    libgstreamer-plugins-bad1.0-dev \
    gstreamer1.0-plugins-base \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-libav \
    gstreamer1.0-tools \
    libsrt-dev
```

##### Verify installation of srtsink and opencv with Gstreamer support

```bash
gst-inspect-1.0 srt
# should not output no element found
```

```bash
python3 -c "import cv2;print(cv2.getBuildInformation())"
## Should show YES on Gsreamer support under Video I/O section
```



### Install pytorch for NVIDIA Jetson


###### Install cuDSS, specific for jetpack >2.0, req for torch to run with GPU acceleration
```bash
wget https://developer.download.nvidia.com/compute/cudss/0.7.1/local_installers/cudss-local-tegra-repo-ubuntu2204-0.7.1_0.7.1-1_arm64.deb
sudo dpkg -i cudss-local-tegra-repo-ubuntu2204-0.7.1_0.7.1-1_arm64.deb
sudo cp /var/cudss-local-tegra-repo-ubuntu2204-0.7.1/cudss-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cudss
```

###### Create a venv for torch and its deps:

```bash
python3 -m venv --system-site-packages ~/jetpipe/perception_venv
source perception_venv/bin/activate
```

###### Install Pytorch (inside venv)
```bash
pip3 install torch torchvision torchaudio --index-url https://pypi.jetson-ai-lab.io/jp6/cu126
```
###### Install Ultralytics with no dependesies:
```bash
pip3 install ultralytics --no-deps
```