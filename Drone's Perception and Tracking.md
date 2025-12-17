
in image_processing.py:

Sub:
- Drone's depth feed
- Drone's RGB feed

Pub:
- rosImage - Processed img ('/front_camera/depth/processed_image')
	- Rcv by ????
- Boolean - personDetected ('/drone/event/personDetected')
	- Rcv by ???
- rosImage - processed img ('/down_camera/depth/processed_image')
	- Rcv by ???
- Track - Person ('/tracking/person')
	- Publishes for every detection, their the bb coordinates (only the bb coordinates are published)
	- Recv by track_server.py in mds_planner




track_server.py:

Sub:
- Track - Person from image_processing.py

Pub:
- Trajectory pub
- OCM_pub
- vehicle_cmd_pub




















```
pipeline_str = (

"srtsrc uri=\"srt://192.168.144.112:9000?latency=1000\" ! "

"tsdemux ! "

"h264parse ! "

"avdec_h264 ! "

"videoconvert ! "

"video/x-raw,format=RGBA ! "

"glupload ! "

"glimagesink name=dronesink sync=false"

)
```

This pipeline works, but with a separate popup window, not integrated with the Qt's GUI, because of Nvidia


