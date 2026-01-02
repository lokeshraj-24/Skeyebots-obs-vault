
Blackboard:
- Shared memories between multiple nodes 
- Accessible thorugh all nodes 

NED: North East Down   ->  x y z 


In py_trees_ros:


##### How the "Tick" Works for py_tree ros

In a standard ROS node, a subscriber waits for a message and triggers a callback immediately. In a Behavior Tree, everything is governed by the **Tick** ⏱️.

1. **The Heartbeat:** The main tree controller sends a "Tick" signal from the root down to the children at a fixed frequency (e.g., 10Hz).
    
2. **The Update:** When the `EventToBlackboard` node receives a tick, it checks if any new ROS messages have arrived since the _last_ tick.
    
3. **The Storage:** If a message arrived, it writes the data to the **Blackboard** 📝.


Important points about BT pulse/tick:

- **The Tick is a Pulse, not a Path:** The tree doesn't "walk" through the code once and wait; it "pulses" (ticks) from the top-down many times per second.
    
- **`RUNNING` is a Status Report:** When a node returns Running, it acts similar to Failure msg in Sequence. The tick is interrupted and the RUNNING message is sent back to the root, irrespective of whether its' in sequence or selector. 
    
- ###### How the priority actually overrites the less priority task in progress:
	- By restarting the tick from the top-left every time, the tree treats your nodes like a **Priority List**.
    - Even if "Move to Location" is `RUNNING`, the next pulse _must_ pass through the "RTL Check" first.
    - If that check suddenly passes, the "Move" node is instantly bypassed (interrupted).


Behaviour tree's behaviour:
- Memory:
	- Parent node's memory determines its reactivity towards its child nodes.
	- If mem=False:
		- --- Parent node forgets anything abt its children and every time its ticked, it'll check from left to right again like in the first time
	- if mem=True:
		- --- Parent node remembers the output from last tick, in case few children were successful and one of it was running in last tick, the parent node remembers that and skips all the successful children from prev ticks and jumps straight into the running one and ticks that





Doubts:
- Why does the Track server uses Followpath action to set goal
	- Followpath action sets Point3D as goals, but here there are no point to chase, only velocity vector. 
	- --- We're just using the Followpath action as a one/off switch, and not actually using its feedback/Goal paramaters, only the result(boolean).
- What is FromConstant, is it an Action client wrapped by a Behaviour Tree node
	- --- Yes
- Why is Goal not defined in any Action clients, 
	- --- Goal in this case just acts like a on/Off switch 
	- But if thats the case, why is it even defined as Point3D in the first place.
- How exactly does cancellation works, cancel_callbacks n stuff




Ship & vessel detection:
- fishing boats vs naval vs container vs small boats, speed boats
- Data gathering, creating database based on detection
- Bird detection, down up or up down
- Identification of each birds