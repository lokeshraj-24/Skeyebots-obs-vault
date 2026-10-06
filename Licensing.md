

Safe projects:
- PyTorch
- Opencvv
- ROS2
- GStreamer
- numpy


Of concern:
- YOLO
	- Under AGPL-3.0 License, which requires us to make our product open source if we're using YOLO in it, else we'll have to buy a license from them
- H264/265:
	- Requires license to use, but Jetson has licens e to use hardware and software encoding, so if we're using NVDIA Jetson's inbuild harware encoder/decoder then it's covered by Jetson. It's safe to use jetson's own hw encoder: `nvv4l2h264enc`
