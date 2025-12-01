

System Architecture 
![[Pasted image 20251201100520.png]]


Network Architecture:
![[Pasted image 20251201100634.png]]


## Links:

1) Sensor - Edge AI: Stream + UDP (one connection)
2) EdgeAI - CommsLayer(RTSP): Stream + ROS (UDP)
3) Comms Layer - Master AINode: UDP and Stream


### Sensor - EdgeAI: Streeam + UDP

	- one physical connection for both Stream+UDP
	- Required bandwidth: 1.5Gbps (for raw data)
	- Latency: 
		- UDP: <1ms (tentative, to be known by Tonbo)
		- Video: ~~ (tentative, to be known by Tonbo)



### EdgeAI - Communication Layer: ROS(UDP) + RTSP

	- Wireless Connection, NLOS Scenario
	- Required Bandwidth per connection:




| **Frame Type**          | **Processing Time** | **Encode + Net + Decode** | **Total Latency**      |
| ----------------------- | ------------------- | ------------------------- | ---------------------- |
| **Frame 1 (Detection)** | 250 ms              | + 80 ms                   | **~330 ms** (Max Lag)  |
| **Frame 2 (Tracking)**  | 15 ms               | + 80 ms                   | **~95 ms** (Real-time) |
| **Frame 3 (Tracking)**  | 15 ms               | + 80 ms                   | **~95 ms**             |
| **Frame 4 (Tracking)**  | 15 ms               | + 80 ms                   | **~95 ms**             |
| **Frame 5 (Tracking)**  | 15 ms               | + 80 ms                   | **~95 ms**             |

| **Stage**                   | **Time (ms)** | **Description**                              |
| --------------------------- | ------------- | -------------------------------------------- |
| **1. Capture & Threading**  | 5 ms          | Time to grab the "latest" frame from buffer. |
| **2. AI Inference**         | 100-250ms     | The dominant delay. The "Thinking" time.     |
| **3. Drawing & Encoding**   | 20 ms         | Overlaying boxes and H.264 encoding.         |
| **4. Network Transmission** | 30 ms         | 2 Hops over Mesh Radio (Signal travel).      |
| **5. Decoding (Master)**    | 30 ms         | Decompressing the H.264 frame.               |
| **6. Player Buffering**     | **~200 ms**   | **CRITICAL RISK** (See Section 3).           |
| **TOTAL**                   | **~535 ms**   | What the operator actually experiences.      |


| **Quality Setting** | **Bitrate (H.264)** | **Visual Clarity** | **Application**                           |
| ------------------- | ------------------- | ------------------ | ----------------------------------------- |
| **Low**             | 1.5 Mbps            | Blocky artifacts   | Situational Awareness (Wide view)         |
| **Medium**          | **3.0 Mbps**        | **Clear edges**    | **Target Identification (Truck vs Tank)** |
| **High**            | 6.0 Mbps            | Crystal clear      | License Plate Reading (Overkill)          |


| **Metric**             | **Value**               | **Notes**                                       |
| ---------------------- | ----------------------- | ----------------------------------------------- |
| **Input Resolution**   | 1080p (1920x1080)       | Raw capture.                                    |
| **Logic Cycle**        | **1 Detect + 4 Tracks** | The pattern repeats every 5 frames.             |
| **Detection Time**     | 250 ms                  | Occurs on Frame 1, 6, 11...                     |
| **Tracking Time**      | **10 - 15 ms**          | Occurs on Frames 2-5, 7-10... (Using KCF/CSRT). |
| **Average FPS**        | **~17 FPS**             | Math: `5 frames / (250 + 4*10)ms = 17.2 Hz`     |
| **Required Bandwidth** | **2.5 - 3.0 Mbps**      | 15 FPS requires slightly more data than 5 FPS.  |
| Safe Bandwidth range   | 5Mbps                   |                                                 |

### Comms layer - Master AI Node
	- Wired connection
	- Bandwidth: 5Mbps x Nc(No of camera)
	- latency: <1ms


## NODES:

- Edge AI Node
- Communication Node
- Master AI Node


### Edge AI Node:

Compression algo:
- H.264, safer bet
###### H.264 vs H.265 comparison:

| h264                    | h265                          |
| ----------------------- | ----------------------------- |
| Less computational load | High computational load       |
| Good for 1080p/720p     | For 4k, 8k                    |
| Low latency             | High latency compared to h264 |
| Better support for RTSP | Mixed supp with RTSP server   |
	Conclusion: 
		-H264 is safer option while using 1080p
		- Only thing h265 does better is reducing the bandwidth almost ~50% less than h264, at the cost of:
			- Higher latency
			- Need for GPU, hardware encoding



Input:
- 1080p 30FPS, raw data (~1.5Gbps)

Functions:
- Decoding?
- AI-Inference**
- UDP communication (bi-directional)
- ROS listener 
- Video encoding





### Communication Layer:

Input:
- h264 codec compressed Processed feed x Nc (no of camera)
- ROS Node (UDP) 

Functions:
- Host RTSP Server:
	- Capture processed feed from Sensors
	- Push the feed to a new RTSP link (no encoding/decoding)
- Host ROS Server:
	- Pub/Sub to Edge nodes
	- Pub/Sub to Master AI node



### Master Node:

Input:
- Single physical connection
- h264 codec compressed Processed feed


Functions: 
- Ros Pub/sub to comms layer
	- pub - Cmd to stop tracking 
	- pub - Cmd to switch target
	- sub - Vision_msg: bbox, class id, tracker id
- Corroboration between camera:
	- instructing the second camera where to look when a target from camera 1 is about to move out of frame
	- Communication between multiple sensors, like a network bridge
	- NOT ADVISED FOR RUNNING 2ND 
- Act as Network bridge for comm layer and GCS