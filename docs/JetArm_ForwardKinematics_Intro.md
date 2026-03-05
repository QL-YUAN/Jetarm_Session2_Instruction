# JetArm Forward Kinematics (FK) Demo with ROS

This document introduces a **Forward Kinematics (FK)** demo for the **JetArm robotic arm** using ROS and Python.
The example shows how to:

* Send **servo pulse width commands** to the robot
* Convert servo pulses to **joint angles**
* Compute the **end-effector pose** (position and orientation) using forward kinematics
* Publish servo commands via ROS topics

---

## 1. What Is Forward Kinematics?

**Forward Kinematics (FK)** answers the question:

> *“Given the joint angles of the robot, where is the end effector?”*

In this demo:

* **Input**: Servo pulse width values (0–1000)
* **Process**: Pulse → joint angles → FK computation
* **Output**:

  * End-effector position `(x, y, z)`
  * End-effector orientation (Quaternion and RPY)

---

## 2. Prerequisites

Make sure the following are installed and working:
* JetArm robot powered on and connected

---

## 3. Initialize the ROS Node and FK Solver

```python
rospy.init_node('fk_demo', anonymous=True)
fk = ForwardKinematics(debug=True)
```

* Initializes a ROS node named `fk_demo`
* Creates a `ForwardKinematics` object

---

## 4. Define Servo Pulse Width Values

```python
servo_list = [200, 400, 300, 400, 500]
print("Servo pulse width values:", servo_list)
```

* Each value corresponds to **one servo motor**
* Pulse range is typically **0–1000**
* The order matches the JetArm joint order

---

## 5. Convert Servo Pulses to Joint Angles

```python
angle_value = transform.pulse2angle(servo_list)
print("Servo pulse width values converted to radians:", angle_value)
```

* Converts servo pulse widths into **joint angles (radians)**
* This conversion is required before running FK
* Internally based on JetArm’s servo configuration

---

## 6. Compute Forward Kinematics

```python
res = fk.get_fk(angle_value)
print('Forward kinematics solution - Coordinates:', res[0])
print('Forward kinematics solution - Quaternion:', res[1])
print('Convert quaternion to RPY:', transform.qua2rpy(res[1]))
```

The FK solver returns:

* `res[0]`: End-effector position

  ```text
  [x, y, z]
  ```
* `res[1]`: End-effector orientation (Quaternion)
* Quaternion is converted to **Roll–Pitch–Yaw (RPY)** for easier understanding

---

## 7. Servo Feedback Detection Logic

```python
bus_servo_data_detection = False
j = 1
```

A callback function tracks servo movement feedback:

```python
def bus_servo_data_callback(msg):
    global bus_servo_data_detection, j
    if msg.servo_id == j:
        j += 1
    if msg.servo_id == 5:
        bus_servo_data_detection = True
```

* Confirms servos are responding
* Stops motion once all 5 servos have been triggered

---

## 8. Servo Command Publisher Function

```python
def bus_servo_controls(id=0, position=0, duration=0.0):
    data = SerialServoMove()
    data.servo_id = id
    data.position = position
    data.duration = duration
    bus_servo_pub.publish(data)
```

Parameters:

* `id`: Servo ID (1–5)
* `position`: Pulse width value
* `duration`: Motion time (milliseconds)
*  robot will response when ID 1-5 are all received
---

## 9. ROS Topic Communication

```python
bus_servo_pub = rospy.Publisher(
    '/jetarm_sdk/serial_servo/move',
    SerialServoMove,
    queue_size=1
)

bus_servo_sub = rospy.Subscriber(
    '/jetarm_sdk/serial_servo/move',
    SerialServoMove,
    bus_servo_data_callback
)
```

* Publishes servo movement commands
* Subscribes to feedback from the same topic

---

## 10. Execute Servo Movements

```python
while True:
    rospy.wait_for_service('/kinematics/set_pose_target')
    bus_servo_controls(id=j, position=servo_list[j-1], duration=500)
    rospy.sleep(0.25)
    if bus_servo_data_detection:
        break
```

* Sends servo commands **one by one**
* Small delay prevents bus overload
* Loop exits once all servos have moved

---

## 11. Summary

This FK demo demonstrates a complete workflow:

1. Define servo pulse values
2. Convert pulses → joint angles
3. Compute forward kinematics
4. Publish servo commands via ROS
5. Verify movement using feedback

It is ideal for:

* Understanding JetArm kinematics
* Verifying servo configurations
* Teaching FK concepts in robotics courses

---

## Running the Script
0. Python File 
-(ROS FK Node): fk.py & fk_exercise.py.
-Copy file not existing *.py to:~/jetarm/src/jetarm_example/src/kinematics_demo
1. For Robot Control 
- Firstly, start a terminal window and run (you can bypass this if you already have this node. ): 

```
exec bash
sudo systemctl stop jetarm_bringup.service
roslaunch jetarm_bringup base.launch
```

To run this IK node, start a terminal window and run:

```
exec bash
cd ~/jetarm/src/jetarm_example/src/kinematics_demo
python3 fk.py
```

## fk.py

```
#!/usr/bin/env python3
# encoding: utf-8
# Date:2021/08/12
import rospy
import signal
import jetarm_kinematics.transform as transform
from jetarm_kinematics.forward_kinematics import ForwardKinematics
from jetarm_kinematics.inverse_kinematics import get_ik, get_position_ik, set_link, get_link, set_joint_range, get_joint_range
from hiwonder_interfaces.msg import SerialServoMove

from jetarm_sdk import bus_servo_control
from hiwonder_interfaces.msg import MultiRawIdPosDur

rospy.init_node('fk_demo', anonymous=True)  # Initialize the node
fk = ForwardKinematics(debug=True)
print("Servo pulse width values:")
servo_list = [200,400,300,400,500]
print(servo_list)  

print("Servo pulse width values converted to radians:")
pulse = transform.pulse2angle(servo_list)  # Convert servo pulse width values to radians
print(pulse)

res = fk.get_fk(pulse)  # Get forward kinematics solution
print('Forward kinematics solution - Coordinates:', res[0])
print('Forward kinematics solution - Quaternion:', res[1])
print('Convert quaternion to RPY:', transform.qua2rpy(res[1]))

bus_servo_data_detection = False
bus_servo_id_detection = False
j = 1
# Bus servo data callback function
def bus_servo_data_callback(msg):
    global bus_servo_data_detection, j
    #print(msg)
    if msg.servo_id == j:
        j += 1
    if msg.servo_id == 5:  # Check if the ID of this topic is empty
        bus_servo_data_detection = True

def bus_servo_controls(id=0, position=0, duration=0.0):
    # Set the bus servo message type
    data = SerialServoMove()
    data.servo_id = id  # Bus servo ID    
    data.position = position  # Bus servo angle [0-1000]
    data.duration = duration  # Bus servo running time
    bus_servo_pub.publish(data)  # Publish data

# Publish bus servo topic
bus_servo_pub = rospy.Publisher('/jetarm_sdk/serial_servo/move', SerialServoMove, queue_size=1)
# Subscribe to bus servo topic
bus_servo_sub = rospy.Subscriber('/jetarm_sdk/serial_servo/move', SerialServoMove, bus_servo_data_callback)

while True:
    rospy.wait_for_service('/kinematics/set_pose_target')
    bus_servo_controls(id=j, position=servo_list[j-1], duration=500)  # Publish data
    rospy.sleep(0.25)
    if bus_servo_data_detection:
        break
```


