
Actions Points:
- SWITCH TO SRT
	- RTSP is bound to fail in wireless network, esp in long dist(reason for Drone and Tonbo glitchy feed)


#### How SRT works that differs from RTSP:
SRT takes the raw speed of UDP and adds a smart "control layer" that mimics TCP’s reliability without TCP’s heaviness or latency spikes.


**Disadvantages in using RTSP:**
- RTSP uses either: RTP/UDP or RTP/TCP, and both fail at long distance
- Wireless links always introduces burst packet loss, so any kind of wireless communication needs to have really good counter measure. packet loss happens due to these facts:
	- Facind
	- Multipath
	- Interference
	- Jitter
	- micro dropouts
- RTSP CANNOT RECOVER FROM BURST LOSSES, so vid freezes + choppiness
- RTSP does not have Adaptive Retransmission, so if a packet is lost, it results in missing frames
- RTSP was designed for LAN, not WAN
- RTSP is meant for CCTV cams, network within same bulding
- 

**How SRT fixes these:**
- Uses UDP + smart re transmission + jitter buffering + packet loss recovery + rate control
- SRT can handle:
	- 20% packet loss (RTSP dies at 2%)
	- high jitter
	- Rapid bandwidth changes
	- Micro outages up to 500-800ms
	- Long distance

**Ket features:**
1) ARQ (Automatic Repeat reQuest): only resends lost packets(UDP drops it and TCP stalls the stream)
2) Adaptive jitter buffer: Absobs unstable periods in wireless
3) Bonding friendly when we multiple relay nodes
4) AES encryption built in
5) Lower latency even with recovery, but needs to be sent:
	- latency = 50ms (good link)
	- latency = 200ms (instable, longrange)
	- latency = 500ms (very unstable multi-hop)

### Additional tricks (for long-range deployments)

To make it bulletproof:

- Reduce bitrate to match wireless throughput (e.g., 2–4 Mbps)
    
- Use constant bitrate (CBR) or capped VBR
    
- Limit resolution (720p or 480p for unreliable links)
    
- Tune SRT latency (200–400 ms is enough)




SRT pipeline code:
- no need to create a speerate server and moint the feed into it, SRT is more close to UDP than RTSP
- key-int-max -> determines how often IDR-keyframes are sent, master frames which are used to recover a feed. A value of 15 with FPS 30 means, 1 frame is sent every 0.5sec (kInt/FPS)
	- Drawbacks: takes more bitrate
- Latency = determines the wait period of srt to regain a lost package before giving up



jetson SRT:
- latency set by SRT : 500 ms
- Experienced latency: 1000 +- 100 ms
- Gstin pipeline:
```
gst_in = (
    f"rtspsrc location={RTSP_URL} drop-on-latency=true ! "
    "rtph265depay ! "
    "h265parse ! "
    "nvv4l2decoder ! "
    f"nvvidconv ! video/x-raw(memory:NVMM) ! " 
    f"nvvidconv ! video/x-raw, format=BGRx ! " # Download from NVMM to CPU memory
    "videoconvert ! video/x-raw, format=BGR ! "
    "appsink drop=true sync=false"
)
```

```
gst_out = (
    f"appsrc ! "
    f"video/x-raw, format=BGR, width={640}, height={480}, framerate=30/1 ! " # Specs must match exactly
    f"videoconvert ! "
    f"x264enc tune=zerolatency speed-preset=ultrafast key-int-max=30 ! " # Re-encode
    f"mpegtsmux alignment=7 ! "
    f"srtsink uri=srt://:9000 mode=listener latency=500"
)
```



To play the video with no latency:
```
ffplay -fflags nobuffer -flags low_delay -framedrop -strict experimental srt://192.168.144.112:9000
```

For Ground unit comms layer:
- Two options to send feed from Edge AI to Comms layer:
	- Create a seperate UDP port for each Edge node, to the comms layer
	- Have one single port through which all 5 cameras are streaming the video



Client side terminal gst code:

```
gst-launch-1.0 -v   srtsrc uri=[srt://192.168.144.112:9000](srt://192.168.144.112:9000) wait-for-connection=false !   tsdemux name=demux   demux. ! queue leaky=downstream ! h264parse ! avdec_h264 max-threads=4 !   videoconvert n-threads=4 !   autovideosink sync=false


gst-launch-1.0 -v   srtsrc uri=srt://192.168.144.112:9000 latency=1000 wait-for-connection=false !   tsdemux name=demux   demux. ! queue leaky=downstream ! h264parse ! avdec_h264 max-threads=4 !   videoconvert n-threads=4 !   autovideosink sync=false
```

```
source: "gst-pipeline: " + 
		"srtsrc uri=\"srt://192.168.144.112:9000?latency=1000\" ! " + 
		"tsdemux ! " + 
		"h264parse ! " + 
		"avdec_h264 ! " + 
		"videoconvert ! " + 
		"qmlvideosink sync=false"
```



Possible routes to check:
- [ ] Integrate Gst pipeline directly into the main.py file, which integrates qml file
	- [ ] Gst pipeline directly pushes the frame into the qml app
	- [ ] a local opencv capture setup in the main.py file, which captures the srt feed (tested and verified), and pushes the numpy frames to the qml app



VideoOutput {
    id: videoOutput
    objectName: "videoOutput" // <--- This MUST match the Python findChild string
    anchors.fill: parent
}


test_pipeline for srt:

```
test_pipeline_str = (
"videotestsrc ! "
"videoconvert ! "
"video/x-raw,format=RGBA ! "
"appsink name=mysink emit-signals=True sync=False max-buffers=1 drop=True"
)
```


Packages needed to add Gstreamer pipeline in qml code:

```
sudo apt install libqt5multimedia5-plugins gstreamer1.0-qt5
```



TRex VLR

Avenger