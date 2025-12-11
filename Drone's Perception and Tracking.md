
in image_processing.py:

Sub:
- Drone's depth feed
- Drone's RGB feed

Pub:
- rosImage - Processed img ('/front_camera/depth/processed_image')
- Boolean - personDetected ('/drone/event/personDetected')
- rosImage - processed img ('/down_camera/depth/processed_image')
- Track - Person ('/tracking/person')
	- Publishes for every detection, their the bb coordinates (only the bb coordinates are published)




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