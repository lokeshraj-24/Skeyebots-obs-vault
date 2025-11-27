
### Example: position telemetry (Pixhawk → Jetson)

1. **PX4 publishes** a uORB message like `vehicle_odometry` internally.
    
2. The **Micro XRCE-DDS Client** inside PX4 subscribes to that uORB topic.
    
3. It **serializes** the message into XRCE packets and sends them over **UART**.
    
4. On the Jetson, the **Micro XRCE-DDS Agent** receives those bytes, **deserializes** them into standard DDS messages.
    
5. DDS middleware automatically exposes those as ROS 2 topics (since ROS 2 uses DDS underneath).
    
6. Your ROS 2 node can now `ros2 topic echo /fmu/vehicle_odometry/out`.
    

At no point does ROS 2 directly “talk” to uORB; the **Agent** does the translation.

---

### Example: sending a setpoint (Jetson → Pixhawk)

1. Your ROS 2 node publishes to `/fmu/trajectory_setpoint/in` (a normal ROS 2 topic).
    
2. DDS delivers that to the **Micro XRCE-DDS Agent**.
    
3. The Agent serializes it into XRCE packets and pushes them through UART.
    
4. The **Micro XRCE-DDS Client** in PX4 receives and converts it back into a uORB message (`trajectory_setpoint` topic inside PX4).
    
5. PX4’s offboard controller picks it up and uses it for control.