
github Token: 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

Easy_write: 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


![[Pasted image 20260216154040.png]]


# Tasks:

- [x] Switch to SRT
	- [x] Research on SRT server - python script
- [ ] Develop CNN Model (~3-4 weeks)
	- Challenges to account for
		-  Humans taking less px
		-  Occlusion(person getting behind an object and comes back)
		-  Humans getting merged with bg
		-  Different human postures 
		-  Group of men
		-  Vehicles and stuff
		-  Animals
		-  Disabling IR when it's too sunny
	- Work in reinforcement learning 
		- Reinforcement learning having stochastic ***
- [x] Drone's Perception Pipeline
	- [x] Develop and verify SRT stream in 2x2 frame
	- [x] Send Data frames along with mdata for tracking
	- [x] Make the GUI touch responsive, let the user select id by just clicked the bbox
		- [x] Test it in Jetson
- [ ] Drone's Work:
	- [x] Drone's Perception:
		- [x] Switch to srt from drone to gui
		- [x] Ros communication between both
		- [x] Send and receive metadata along with frames
		- [x] Select Object to track
			- [x] Make GUI touch responsive
			- [x] Send the selected ID back to Drone
	- [x] Setup codebase in my laptop 
	- [ ] Implement Tracking in drone (~3 weeks)
		- [x] Get Sim's camera feed in the GUI for testing
		- [x] Add Human models
		- [x] Add Detection Algorithm in the simulation
		- [ ] Add track_server action in the behaviour planner (blocked till AK merges PR)
		- [ ] Develop Tracking algorithm
			- [[Drone's Perception and Tracking]] 
			- [x] Research, fix on initial iteration:  
			- [ ] Implement 1st iteration 
				- [ ] Implement Gimbal control
				- [ ] Implement Drone control
			- [ ] Integrate track_server in behaviour planner
		- [ ] Add gimbal movement control in simulation
		- [ ] Finish testing in simulation
		- [ ] Test it in field
- [x] GUI
	- [x] Add Select object tracking pipeline in gui
		- [x] Send webcam feed along with json msg and receive it in client side
		- [x] Find a way to receive the data in qml by setting separate property
		- [x] Integrate both and final testing
	- [x] Make the GUI's camera region touch responsive
		- [x] Find a way to get the mouse clicked co-ordinates within QML
		- [x] Verify the capture of mouse's click co-ordinate in main.py
		- [x] Convert received bbox to a one shot object
		- [x] Match the mouseClick value to id
		- [x] Send the id to mds_drone
			- [x] Prepare a udp comms b/w mds gui and drone
		- [x] Test in Jetson:
			- [x] Verify a stable srt pipeline from jetson
			- [x] Implement same pipeline in the mdata pipeline
			- [x] Test everything all-together and verify
- [ ] Analytics (~2 weeks)
	- [ ] Store processed video locally
		- [ ] access to that stored file to C2 device
	- [ ] Produce metadata realtime on each inference modules
	- [ ] Gather all metadata -> produce final inference (happens in C2 device)
		- [ ] Possible option to go with Analytics LLM
- [ ] Ground Unit Perception (~2 weeks)
	- [ ] Tonbo custom tracking (Angular kalman)
	- [ ] Lockin to the selected target's centre point with minimal delay and initiate tracking
- [ ] Ground Unit Pipeline (~1.5 weeks)
	- [ ] Frames and commands comms from AI Node to GCS
	- [ ] Get ~0sec delay using the SRI convertor
	- [ ] Add No option - back to sweep
		- [ ] To be able to ignore the defections that was prompted No before
	- [ ] Corroboration between multiple cameras
- [ ] Setup AI Node (~1-3 days)
- [ ] Drone:
	- [ ] TRN for GPS spoofing




Ship & vessel detection:
- fishing boats vs naval vs container vs small boats, speed boats
- Data gathering, creating database based on detection
- Bird detection, down up or up down
- Identification of each birds



