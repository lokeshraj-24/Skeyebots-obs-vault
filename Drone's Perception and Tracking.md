
In image_processing.py:

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




## Tracking

-> Hybrid approach of both Gimbal and drone's control

##### Algo 1 (Hybrid) :
- Set a specific pan/tilt as home position for gimbal
- Gimbal alone tracks the person
- Drone keeps track of drone's current pan/tilt, and moves slowly so as to bring the gimbal 

##### Algo 2 (Drone alone)
- Gimbal is locked on a specific pan/tilt
- ROI region in a frame
- When human enters the border (20-30% across the border), then move the drone to push the human back into the square

##### Algo 3 (semi-Hybrid):
- Drone in lock mode, only gimbal moves
- Only when gimbal's pan/tilt angle goe beyond a certain threshold, inform drone to move in the respective direction



Control Modes:

###### Manual:
- RC remote/joystick
- Through MAVLink


##### Auto:
- Automatic takeoff with predetermined parameter
- Actuator's parameters are automoatic
- MAVLink


###### Offboard:
- Manual but Jetson's controlled
- Gives instruction on vx, vy



Tracking Algo: HYBRID MODE:

- Gimbal cam:
	- in lock mode
		- The movement is with respect to world frame, not drone
		- If drone is prompted to 45 deg, and if the drone yaw for certain angle, that doesn't affect the gimbal's final position. 
	- Velocity based tracking:
		- Custom PID?
			- if so: Higher derivative. 
		- Vc​=−λ.Le+​e(t)
			- λ -> gain constant, how aggresively we want it to catch up to the target (pref less as we're high up in the air)
			- Le -> jacobian Constant.
				- used to convert pixel difference to degree
				- Basically a dynamic constant, that accounts for changing zoom level, magnitude of error, distance between drone and target. 
			- e(t) -> error in pixel difference
	- Setpoint based tracking:
		- Need focal length/FOV
		- Calculations to be done on Jetson side
	- Moves only with respect to the detection
- Drone
	- Only looks at camera's angle
	- Has a set HOME position for cam
	- If cam.pan or tilt not home position, with 1deg buffer, then move drone to bring the camera back to home position
		- e(pan) > 0 -> move left and vice versa
		- e(tilt) < 0 -> move forward and vice versa



Formula for converting Pixel error to velocity values (Both drone and gimbal):

## Gimbal Target Tracking Controller

For this use case, a **PD controller with a deadband** is usually the sweet spot — pure P causes oscillation/hunting, adding D damps it nicely, and I-term is often counterproductive for visual tracking (it winds up when the target is lost or at the edge).

---

### Recommended Formula

```python
# At each 50Hz tick (dt = 0.02s)

# --- Deadband (lock zone) ---
THRESHOLD = 5  # pixels, tune this

if abs(e_x) < THRESHOLD and abs(e_y) < THRESHOLD:
    vel_yaw   = 0
    vel_pitch = 0
else:
    # --- Derivative (rate of error change) ---
    de_x = (e_x - prev_e_x) / dt
    de_y = (e_y - prev_e_y) / dt

    # --- PD Output ---
    vel_yaw   = Kp * e_x + Kd * de_x
    vel_pitch = Kp * e_y + Kd * de_y

    # --- Clamp to max velocity ---
    vel_yaw   = clamp(vel_yaw,   -VEL_MAX, VEL_MAX)
    vel_pitch = clamp(vel_pitch, -VEL_MAX, VEL_MAX)

prev_e_x = e_x
prev_e_y = e_y
```

---

### Why each term:

**Kp (Proportional)** — the core driver. Large error → fast movement, small error → slow movement. Alone it causes steady-state oscillation (overshoot and hunt around the target).

**Kd (Derivative)** — braking force. It sees the _rate_ at which error is closing and slows the gimbal down before it overshoots. This is what gives you smooth lock-on without struggle. Think of it as predictive braking.

**No I-term** — with a moving target at 50Hz, integral windup causes more harm than good. The only reason to add I would be if your gimbal has mechanical bias/friction causing a consistent offset, and even then a small clamped I-term is enough.

---

### Starting Tuning Values

|Parameter|Suggested Start|Effect of increasing|
|---|---|---|
|`Kp`|`0.05 – 0.15`|Faster response, more overshoot|
|`Kd`|`0.005 – 0.02`|Less overshoot, more damping|
|`THRESHOLD`|`5 – 15 px`|Larger = locks sooner but less precise|
|`VEL_MAX`|hardware limit|Prevents mechanical damage|

Tune `Kp` first until it tracks well but oscillates slightly, then increase `Kd` until the oscillation dies without making it feel sluggish.

---

### Optional Improvement: Normalized Error

Instead of raw pixels, normalize by frame dimensions so your gains are resolution-independent:

```python
e_x_norm = e_x / (frame_width  / 2)   # range: -1.0 to +1.0
e_y_norm = e_y / (frame_height / 2)
```

Then `Kp` becomes intuitive: `Kp = 0.5` means "at full frame edge, command 50% of max velocity."

---

### Flow Summary

```
Read e(x), e(y)
    │
    ▼
Within threshold? ──YES──► vel = 0
    │
    NO
    ▼
Compute derivative (de/dt)
    ▼
vel = Kp·e + Kd·(de/dt)
    ▼
Clamp to [-VEL_MAX, VEL_MAX]
    ▼
Send to gimbal, store prev_e
```

This will give you fast acquisition, smooth deceleration as it closes in, clean lock-on with the deadband, and automatic re-engagement when the target moves — all without noticeable hunting.