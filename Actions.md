
used for a long term runnign process
- Does not keep the client on hold until an action is done. Just takes the goal and starts working on it, while also being open to working on other things
- Can also send feedback when requested in the middle of performing a certain action


To create Action server:
- Inherit the class Node from rclpy.node, and give a name to the constructor: super().__init__('track_action')
- To define action Server:
```
self._action_server = ActionServer(
	self,
	FollowPath, # The Action Type
	'action_name', # The Action name 
	self.execute_callback, # The function to run when a goal is set, where the main action takes place
	self.cancel_callback, # Called when cancellation is requested
	) 
```

##### Execture_callback loop:
Inside the loop, you would typically:
1. Check if the goal is still active.
2. Update your progress.
3. Create a `Feedback` object and call `goal_handle.publish_feedback(feedback)`.
4. Sleep for a bit (to simulate time passing).

This function takes one argument called goal_handle.
- It is used to check if theh goal is active: `goal_handle.is_active`
- It's also how we publish ffeedback while execute_callback is constantly running and controlling the code. Feedback is published by: `goal_handle.publish_feedback()`