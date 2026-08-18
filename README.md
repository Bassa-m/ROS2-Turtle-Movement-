# ROS2-Turtle-Movement

Practical exploration of ROS 2 Humble using Ubuntu 22.04, covering topic communication and turtlesim robot control.

## ROS 2 Turtle Control

## Overview

This repository documents my practice with **ROS 2 Humble on Ubuntu 22.04 using WSL**. I explored the basics of ROS 2 communication and practiced controlling a simulated turtle.

The work covers two main areas:
- Publishing and receiving messages using ROS 2 topics.
- Controlling a turtlesim turtle to draw a square.

---

## Part 1: Publishing a Message in ROS 2

I practiced publishing and receiving messages using ROS 2 Humble through Ubuntu.

### Steps

First, I opened Ubuntu and sourced the ROS 2 Humble environment:
```bash
source /opt/ros/humble/setup.bash

```
I then verified that ROS 2 Humble was active:
```bash
echo $ROS_DISTRO

```
The output was:
```text
humble

```
In the same terminal, I published a message to the /chatter topic:
```bash
ros2 topic pub /chatter std_msgs/msg/String "{data: 'Welcome ROS 2'}"

```
I then opened a second Ubuntu terminal and sourced ROS 2 Humble again:
```bash
source /opt/ros/humble/setup.bash

```
To receive and display the published message, I used:
```bash
ros2 topic echo /chatter

```
### Result
<img width="1482" height="746" alt="Image 2026-08-18 at 8 11 16 PM" src="https://github.com/user-attachments/assets/e6f439e1-ea94-41ea-8c23-ca76b5627cd8" />

## Part 2: Controlling Turtlesim
I then practiced controlling a simulated robot using the ROS 2 turtlesim package.
### 1. Starting Turtlesim
I launched the turtlesim simulator using:
```bash
ros2 run turtlesim turtlesim_node

```
### 2. Check ROS 2 Topics
I checked the available topics using:
```bash
ros2 topic list

```
The main topics used for controlling and monitoring the turtle were:
 * /turtle1/cmd_vel: Used to send velocity commands to the turtle.
 * /turtle1/pose: Provides the turtle’s current position and orientation.
### 3. Create the Square Movement Program
I created a Python program called square.py to control the turtle.
The turtle was programmed to:
 1. Move forward for 2.0 units.
 2. Rotate approximately 90 degrees.
 3. Repeat the movement four times.
 4. Stop after completing the square.
### 4. square.py Code
```python
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist
from turtlesim.msg import Pose
import math

class SquareTurtle(Node):
    def __init__(self):
        super().__init__('square_turtle')
        self.publisher = self.create_publisher(
            Twist,
            '/turtle1/cmd_vel',
            10
        )
        self.pose_subscriber = self.create_subscription(
            Pose,
            '/turtle1/pose',
            self.pose_callback,
            10
        )
        self.timer = self.create_timer(0.05, self.move)
        self.x = 0.0
        self.y = 0.0
        self.theta = 0.0
        self.start_x = None
        self.start_y = None
        self.start_theta = None
        self.state = 'forward'
        self.side_count = 0

    def pose_callback(self, pose):
        self.x = pose.x
        self.y = pose.y
        self.theta = pose.theta

    def normalize_angle(self, angle):
        return math.atan2(math.sin(angle), math.cos(angle))

    def move(self):
        msg = Twist()
        if self.start_x is None:
            self.start_x = self.x
            self.start_y = self.y
            self.start_theta = self.theta

        if self.state == 'forward':
            msg.linear.x = 1.0
            msg.angular.z = 0.0
            distance = math.sqrt(
                (self.x - self.start_x) ** 2 +
                (self.y - self.start_y) ** 2
            )
            if distance >= 2.0:
                msg.linear.x = 0.0
                self.start_theta = self.theta
                self.state = 'turn'

        elif self.state == 'turn':
            msg.linear.x = 0.0
            msg.angular.z = 1.5
            turned = self.normalize_angle(
                self.theta - self.start_theta
            )
            if abs(turned) >= math.pi / 2:
                msg.angular.z = 0.0
                self.side_count += 1
                if self.side_count >= 4:
                    self.state = 'stop'
                else:
                    self.start_x = self.x
                    self.start_y = self.y
                    self.start_theta = self.theta
                    self.state = 'forward'

        elif self.state == 'stop':
            msg.linear.x = 0.0
            msg.angular.z = 0.0

        self.publisher.publish(msg)

def main(args=None):
    rclpy.init(args=args)
    node = SquareTurtle()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()

```
### 5. Run the Program
After starting turtlesim, I opened another Ubuntu terminal and sourced ROS 2 Humble:
```bash
source /opt/ros/humble/setup.bash
python3 square.py

```
### Result
<img width="1600" height="826" alt="Image 2026-08-18" src="https://github.com/user-attachments/assets/f3fdb30e-7431-4ab1-b2dd-b769839106e0" />

## Problems Encountered and Solutions
 1. **Turtlesim Window Was Not Appearing Correctly**
 * *Solution:* Shut down WSL from PowerShell using wsl --shutdown, then restarted Ubuntu.
 2. **Python NameError**
 * *Solution:* Corrected the main guard syntax to:
   ```python
   if __name__ == '__main__':
       main()
   
   ```
 3. **Turtle Path Inaccuracies**
 * *Solution:* Used /turtle1/pose feedback and adjusted linear velocity (msg.linear.x = 1.0) for smoother control.
## What I Learned
Through this practice, I learned how to:
 * Set up and use ROS 2 Humble in Ubuntu through WSL.
 * Publish and receive messages using ROS 2 topics.
 * Launch and work with turtlesim.
 * Control a simulated robot using velocity commands and pose feedback.
 * Debug Python and ROS 2 errors effectively.
```

```
