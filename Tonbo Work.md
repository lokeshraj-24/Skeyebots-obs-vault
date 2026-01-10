
## Reducing delay

- Native delay is close to 200-500ms.
- Pulling feed directly through ffplay results in delay

To do:
- [ ] ###### Test with custom Gstreamer pipeline
	- [x] Test the stream with custom gst pipeline independently without opencv, through terminal
	-   POSITIVE: Getting the feed with very very less delay. used gst pipeline in the terminal:
	-  `rtspsrc location=rtsp://192.168.2.42:9001/video latency=0 ! rtph264depay ! h264parse ! avdec_h264 ! videoconvert ! autovideosink sync=false
		- [x] Once that works, try that same pipeline in the py script with opencv
		- [x] Integrate with existing Tonbo Integration code
		- [x] Check the delay with and without inf
			- noted a slight 500ms additinal delay with the stream after integration 
		- [x] Fix the god forsaken cocksucking fking delay again suigsklksieu
			- Turned out to be the same issue, using the same parameters fixed it, fuck me
		- [ ] Ditch RTSP and switch to srt 
- ###### Test with custom ffmpeg pipeline, inside opencv
	- Not necessary



```
gst-launch-1.0 rtspsrc location=rtsp://192.168.2.71:9001/video latency=0 ! rtph264depay ! h264parse ! avdec_h264 ! videoconvert ! autovideosink sync=false
```


```
gst-launch-1.0 -v srtsrc uri="srt://192.168.2.195:8555?mode=caller&latency=50" ! tsdemux ! h264parse ! nvh264dec ! videoconvert ! fpsdisplaysink sync=false
```



Issues faced:

- Srt integration into the pipeline
	- Just using the cv2.VideoWriter did not work well, it backstabbed me and ended up fucking the entire pipeline
	- Had to reinvent the wheel, build a srt server from ground up to make it behave like an rtsp server

more issues:
- SRT server in main loop just runs two concurrent threads which constantly get EO/IR frames and pushed it into the server irrespective of if anyone's watching. 
	- Need to make it event based similar to RTSP, 
	- Add streamid, so multiple stream can be broadcasted through same port

- Added threads in the incoming RTSP parsing, so it constantly updates eo/ir frames, which is later easily pulled by get_frame() function
	- When model is initialised outside the thread, its not able to use that same model for processing. All the RTSPFeed function had to be run on same process and thread
	- UPDATE: Able to get the feed with threaded RTSP approach ---  Issues founded and fixed:
		- Called get_eo_frame() requesting before the pipeline was een initialised (which takes 1-2sec)
		- get_frame(), reading a variable which could be undergoing writing operation from a seperate thread. 
	- Its working fine when the get_frame() is called in the main loop, but breaks when we run the get_frame() inside another thread.
	- It's working fine when run without inference, confirming **the threading issue is caused purely by the use of CUDA context**
	- Verified by only processing EO frames and not IR frames, the issue came back