# ROS2-Turtle-Movement
Practical exploration of ROS 2 Humble using Ubuntu 22.04, covering topic communication and turtlesim robot control.

## ROS 2 Turtle Control

## Overview


This repository documents my practice with **ROS 2 Humble on Ubuntu 22.04 using WSL**. I explored the basics of ROS 2 communication and practiced controlling a simulated turtle usingurtle Control

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
``` bash
echo $ROS_DISTRO
```

The output was:
```
humble
``` 
In the same terminal, I published a message to the /chatter topic:
```
ros2 topic pub /chatter std_msgs/msg/String "{data: 'Welcome ROS 2'}"
``` 
I then opened a second Ubuntu terminal and sourced ROS 2 Humble again:
```
source /opt/ros/humble/setup.bash
```
To receive and display the published message, I used:
```
ros2 topic echo /chatter
```

Result
> The message “Welcome ROS 2” was successfully published and received through the /chatter topic.
> <img width="1482" height="746" alt="Screenshot 2026-08-12 024219" src="https://github.com/user-attachments/assets/fd5903c5-1961-4078-95ac-6e92d44763c4" />


⸻


## Part 2: Controlling Turtlesim
I then practiced controlling a simulated robot using the ROS 2 turtlesim package.
### 1. Starting Turtlesim
I launched the turtlesim simulator using:
```
ros2 run turtlesim turtlesim_node
```
--- 
### 2. Check ROS 2 Nodes
I checked the available ROS 2 nodes using:
```
ros2 topic list
```
--- 
## 3. Check Available Topics
I also checked the available topics using:
```bash 
ros2 topic list
```
The main topics used for controlling and monitoring the turtle were:
```bash 
/turtle1/cmd_vel
/turtle1/pose
/turtle1/cmd_vel is used to send velocity commands to the turtle.
/turtle1/pose provides the turtle’s current position and orientation.
```
--- 
### 4. Create the Square Movement Program
I created a Python program called square.py to control the turtle.
The program uses:

- /turtle1/cmd_vel to control the turtle’s movement.
- /turtle1/pose to read the turtle’s current position and orientation.
The turtle was programmed to:
1- Move forward for 2.0 units.
2- Rotate approximately 90 degrees.
3- Repeat the movement four times.
4- Stop after completing the square.
  
The program uses the turtle’s actual position and orientation instead of relying only on fixed time intervals.

--- 
## 5. square.py Code
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
            msg.linear.x = 1.5
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

### 6. Run the Program

After starting turtlesim, I opened another Ubuntu terminal and sourced ROS 2 Humble:
```
source /opt/ros/humble/setup.bash
```

I then ran the Python program:
```
python3 square.py
```
The turtle moved forward, rotated approximately 90 degrees, and repeated the process four times before stopping.

 --- 
### Result
The turtle successfully completed a regular square path and stopped after completing all four sides.

<img width="1917" height="990" alt="Screenshot 2026-08-12 055412" src="https://github.com/user-attachments/assets/b232ed3a-f008-4616-9443-26967ef98d56" />


--- 



https://github.com/user-attachments/assets/9ca57360-5b52-443d-959d-9adbd82ee580





⸻


Problems Encountered and Solutions
1. Turtlesim Window Was Not Appearing Correctly
The turtlesim window appeared in the application bar but did not open normally.
I solved this by shutting down WSL from PowerShell:
```
wsl --shutdown
```
After restarting Ubuntu and launching turtlesim again, the window appeared correctly.
3. ROS 2 Environment Was Not Active
Some ROS 2 commands require the ROS 2 environment to be sourced first.
I used:
```
source /opt/ros/humble/setup.bash
```
I then verified the installation using:
```
echo $ROS_DISTRO
```
which returned:
```
humble
```
4. Python 
name
Error
While running square.py, I initially received:
```
NameError: name 'name' is not defined
```
The problem was an incorrect Python main statement.
I corrected:
```
if name == 'main':
```
to:
```
if on Ubuntu== '__main__':
    main()
```
6. Turtle Did Not Draw a Proper Square
The first version of the program used fixed time intervals for movement and rotation. This caused the turtle to gradually deviate from the desired path.
I improved the program by using the turtle’s actual position and orientation from:
```
/turtle1/pose
```
The program then used the distance traveled and rotation angle to control the movement more accurately.
8. The Square Was Slightly Irregular
The turtle was initially moving too quickly, which caused small errors when stopping at the end of each side.
I reduced the linear velocity from:
```
msg.linear.x = 1.5
```
to:
```
msg.linear.x = 1.0
```
This made the movement more controlled and resulted in a more regular square.


⸻


What I Learned
Through this practice, I learned how to:
- Set up and use ROS 2 Humble in Ubuntu through WSL.
- Source the ROS 2 environment.
- Publish and receive messages using ROS 2 topics.
- Understand the relationship between topics, publishers, and subscribers.
- Use ROS 2 command-line tools.
- Launch and work with turtlesim.
- Control a simulated robot using velocity commands.
- Read a robot’s position and orientation.
- Debug Python and ROS 2 errors.
