

To run gz simulation in GPU:
```
cd ~/PX4-Autopilot && \
export GZ_SIM_RESOURCE_PATH=$PWD/Tools/simulation/gz:$GZ_SIM_RESOURCE_PATH && \
export PX4_GZ_WORLD=baylands_new && \
export PX4_HOME_LAT=59.910028 && \
export PX4_HOME_LON=10.750444 && \
export PX4_HOME_ALT=0.0 && \
export __NV_PRIME_RENDER_OFFLOAD=1 && \
export __GLX_VENDOR_LIBRARY_NAME=nvidia && \
make px4_sitl gz_x500_mono_cam
```