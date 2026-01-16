
used for a long term runnign process
- Does not keep the client on hold until an action is done. Just takes the goal and starts working on it, while also being open to working on other things
- Can also send feedback when requested in the middle of performing a certain action


#### To create Action server:
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


#### Action client
To initialise an action client, we need
- Action type
- Action Name

```
self._action_client = ActionClient(self, ActionType, 'action_name')
```

To send a goal to the server,
1) Create a Goal message
 ```
 def send_goal(self, distance):
    goal_msg = ActionType.Goal()
    goal_msg.target_distance = distance

    # Wait for the server to be available
    self._action_client.wait_for_server()

    # Send the goal and register the feedback callback
    self._send_goal_future = self._action_client.send_goal_async(
        goal_msg, 
        feedback_callback=self.feedback_callback
    )

    # Tell the client to run 'goal_response_callback' when the server answers
    self._send_goal_future.add_done_callback(self.goal_response_callback)
 ```
 2) Handling server's response
```
def goal_response_callback(self, future):
    goal_handle = future.result()
    if not goal_handle.accepted:
        self.get_logger().info('Goal rejected :(')
        return

    self.get_logger().info('Goal accepted! Waiting for result...')
    
    # Now that it's accepted, we ask for the final result
    self._get_result_future = goal_handle.get_result_async()
    self._get_result_future.add_done_callback(self.get_result_callback)
```

3) Handling feedback and result
```
def feedback_callback(self, feedback_msg):
    # This runs every time the server sends an update
    percent = feedback_msg.feedback.percent_complete
    self.get_logger().info(f'Progress: {percent}%')

def get_result_callback(self, future):
    # This runs once when the task is totally finished
    result = future.result().result
    self.get_logger().info(f'Final Result: {result.success}')
```



#### Action client in BT