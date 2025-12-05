

### Environment setup in U24 with pytorch (GPU enabled), ultralytics, opencv with GStreamer support


```
sudo apt update
```

1) Install Gstreamer:
```
sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \ libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base \ gstreamer1.0-plugins-good gstreamer1.0-plugins-bad \ gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-tools \ gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3 \ gstreamer1.0-qt5 gstreamer1.0-pulseaudio
```

2) Install opencv from apt: 
```
sudo apt install python3-opencv libopencv-dev
```

3) Create conda env with python=3.12 (required for ubuntu 24):

```
conda create -n sk_cv python=3.12
conda activate sk_cv
```

4) Create the link between the conda env and the original opencv with Gstreamer in base env
```
cd $CONDA_PREFIX/lib/python3.12/site-packages/
ln -s /usr/lib/python3/dist-packages/cv2.cpython-312-x86_64-linux-gnu.so cv2.so
```

5) Install dependencies that are required to run opencv in conda
	- ubuntu 24's python is compiled against np<2.0, using np>2.0 will break opencv
```
pip install "numpy<2.0"
pip3 install --pre torch torchvision torchaudio
pip install ultralytics --no-deps
pip install requests pandas seaborn tqdm py-cpuinfo psutil scipy matplotlib pyyaml	  
```

**Terminal code to check opencv with Gstreamer support:**
```
python -c "
import cv2
print(cv2.getBuildInformation())
"
```
