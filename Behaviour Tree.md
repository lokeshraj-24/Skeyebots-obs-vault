
Draw io diagram link: [Link here](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&dark=auto#R%3Cmxfile%3E%3Cdiagram%20name%3D%22Page-1%22%20id%3D%22DHaTkUI9L9x5ZS0G07oA%22%3E5Vpbc9o4FP41PNKxJN94bEnSdDadzS7pNH3qaLACyhrLK4sA%2B%2BtXtiVsGUi5WJg0T6Bj6Vj%2BvnN0LnYPDWfLzxyn068sInEPOtGyh656ELq%2BB%2BRPLlmVEhiEXimZcBqVMlAJRvQ%2FooSOks5pRDJjomAsFjQ1hWOWJGQsDBnmnC3MaU8sNu%2Ba4gnZEIzGON6UfqeRmJbS0HMq%2BS2hk6m%2BM3DUlRnWk5Ugm%2BKILWoidN1DQ86YKP%2FNlkMS5%2BhpXMp1NzuurjfGSSL2WTB%2F%2FPs5fkHZX3d%2FON%2BeweP8Txj3XViqIdEGDJVeJcrYnI%2FJK8oUs5lYafRytSM1ZFxM2YQlOL6upJ84mycRybfoyFE1546xVAqBFD4TIVbKMvBcMCmailmsriqbwHxCxGsPWkEujZWwGRF8JRdyEmNBX8yHx8poJut566X3jEpYoKMtXNOt7NvVdqtVlBtTqyp25J%2FaNipRwdkh%2FKG3zp%2FcJV895us%2FeHr4Q6krBldLY7RSoz15R%2BGJvBuMHUyP%2B9bp2de9QJcwe7tBrcCqoMifazGlgoxSXCC%2BkMFr22O%2F4HiuVvegH0vVn55Y4f6VXv%2FfOdMX%2BlkB40c5AYbpstCjr8t%2Fk%2Fz3nkiWU0FZolXKRy61lhPUrQkXZFl7lk1sp7W4E6pjZ1HFKKAjrdKCEDKOKhg4lujQ4Ldi9cEFWz3wurR62GbsvmiY%2FU5hbjPEXjTMQacwtxkqLxrmTjMS6L11mC0njGDQJT3BBWYygx2ZzNeHuzZTmGBLCuM6RgrT9zyz3LKXw%2Bg6rhU%2FQYNOHGVJRc1P5OhH7UrlJfngQCc5Ne%2FZUUxD74Nn8Atgg98SamvlNDgk1tfITKKPea9JjsYxzjI6NpkwqbR8gAVbPP5XvqfdaiuBNYf0XnG34g4SBryqTUhzprLdlPfd0PRo4DsNYkuVOywGbV%2F9K3vZVOR128cBh2Q%2FF2p4CLwlywOBc5LlwZYsD8CGpnUtfy7T26t7IhXRNCNH5BsyTRjP5AZujsoKtjU2YCMrGDSbsK8Zy2lZgW8XqyvOEpJvIsl3T7lN1PzB2VDbK6s9HrX7GOd4TUhCOBbEGmJew1Pdps%2B3h1ho3SetoYTQ2VAa2EXpNn9ZaAsm0Mw4rMEE1aHVTimzBeCLaa2gHQnHmVorwbvBGXWKc%2FhucHY7xXnwbnDu9D0O2qvUu5Cm3yh5OHPTD%2FjQbAptFEjtMQHtJhQFeI7dagjAwITLXmKhXyj%2FBtUQcMOzoebaRe1M1RDwvQZi1nrxyHKHonBLSyiF7tlQslwz6mM%2Foi%2F6yH%2FgePzPUB1n66BQm3AUrIM9YkKzEtcot49quzUmztLyK9UnusyD%2BemRGjk7IvWQzWY4ibI2wzXYFq8DkxvkOo23OH1rnSVtFr9BBEJ%2B46DwbJm0u9%2BrhaNB%2B5JQQWXk%2BVkcD7bw8hovjo5pX8ph9f112U%2BvPmNH1%2F8D%3C%2Fdiagram%3E%3C%2Fmxfile%3E)


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




