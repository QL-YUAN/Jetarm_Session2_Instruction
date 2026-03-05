# JetArm Inverse Kinematics Demo with ROS

This tutorial demonstrates how to control a **JetArm robot** using **inverse kinematics (IK)** and ROS messages. The example calculates the joint angles for a desired end-effector position and sends commands to the servos.

---

## Prerequisites

Before running this tutorial, ensure you have:

* A JetArm robot connected and powered

---

## Step 1: Initialize the ROS Node

```python
import rospy

# Initialize the ROS node
rospy.init_node('ik_demo', anonymous=True)
```


---

## Step 2: Define the Target End-Effector Position

```python
coordinate = [0.15, 0.0, 0.0]  # x, y, z in meters
print("End effector coordinates:", coordinate)
```

* The `coordinate` list defines the target position for the robot’s end effector in meters.
* Modify these values to set a new target position.

---

## Step 3: Compute Inverse Kinematics (IK)

```python
from jetarm_kinematics.inverse_kinematics import get_ik
from jetarm_kinematics import transform

servo_list = []

res = get_ik(coordinate, 90, [-180, 180])
if res != []:
    for i in range(len(res)):
        print('rpy%s:' % (i + 1), res[i][1])
        pulse = transform.angle2pulse(res[i][0])
        servo_list = pulse[0]
        for j in range(len(pulse)):
            print('Servo pulse width value%s:' % (j + 1), pulse[j])
```

* `get_ik(coordinate, 90, [-180, 180])` calculates possible joint angles for the end-effector to reach the target.

  * `coordinate`: target `[x, y, z]`
  * `90`: rotation of the end effector pitch ange
  * `[-180, 180]`: joint limits
* `transform.angle2pulse()` converts joint angles (degrees) to servo **pulse width values** (0-1000 scale for JetArm).

---

## Step 4: Define a Callback for Bus Servo Feedback

```python
from hiwonder_interfaces.msg import SerialServoMove

bus_servo_data_detection = False
j = 1

def bus_servo_data_callback(msg):
    global bus_servo_data_detection, j
    if msg.servo_id == j:
        j += 1
    if msg.servo_id == 5:
        bus_servo_data_detection = True
```

* The callback function listens to servo movements.
* `bus_servo_data_detection` signals when all servo IDs have been processed.

---

## Step 5: Create a Function to Publish Servo Commands

```python
from jetarm_sdk import bus_servo_control

def bus_servo_controls(id=0, position=0, duration=0.0):
    data = SerialServoMove()
    data.servo_id = id
    data.position = position
    data.duration = duration
    bus_servo_pub.publish(data)
```

* `id`: servo ID (1–5 for JetArm)
* `position`: target position in **pulse width**
* `duration`: motion duration in milliseconds
*  robot will response when ID 1-5 are all received

---

## Step 6: Publish and Subscribe to Servo Topics

```python
bus_servo_pub = rospy.Publisher('/jetarm_sdk/serial_servo/move', SerialServoMove, queue_size=1)
bus_servo_sub = rospy.Subscriber('/jetarm_sdk/serial_servo/move', SerialServoMove, bus_servo_data_callback)
```

* The publisher sends servo commands.
* The subscriber listens to servo movements and triggers callbacks.

---

## Step 7: Send Commands to the Robot

```python
while True:
    rospy.wait_for_service('/kinematics/set_pose_target')
    if servo_list != []:
        bus_servo_controls(id=j, position=int(servo_list[j-1]), duration=500)
        rospy.sleep(0.25)
        if bus_servo_data_detection:
            break
    else:
        break
```

* Waits for the ROS service `/kinematics/set_pose_target` to be ready.
* Iteratively sends servo pulse commands to the robot.
* Stops when all servos have moved.

---

## Notes

* Adjust `coordinate` and `duration` to experiment with different end-effector positions and speeds.
* Ensure your JetArm is powered on and connected before running the script.
* `rospy.sleep(0.25)` gives a small delay between servo commands to avoid conflicts.

---

## Running the Script
0. Python File 
-(ROS IK Node): ik.py & ik_english.py. (check English version in ik_english.py)
-Copy file ik_english.py to:~/jetarm/src/jetarm_example/src/kinematics_demo
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
python3 ik_english.py
```

## ik_english.py

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

rospy.init_node('ik_demo', anonymous=True)  # Initialize the node
print("End effector coordinates:")
coordinate = [0.15, 0.0, 0.0]
print(coordinate)  

servo_list = []

res = get_ik(coordinate, 90, [-180, 180])  # Get inverse kinematics solution
if res != []:
    for i in range(len(res)):
        print('rpy%s:' % (i + 1), res[i][1])  # rpy values corresponding to the solution
        pulse = transform.angle2pulse(res[i][0])  # Convert to servo pulse width values
        servo_list = pulse[0]
        for j in range(len(pulse)):
            print('Servo pulse width value%s:' % (j + 1), pulse[j])

bus_servo_data_detection = False
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
    if servo_list != []:
        bus_servo_controls(id=j, position=int(servo_list[j-1]), duration=500)  # Publish data
        rospy.sleep(0.25)
        if bus_servo_data_detection:
            break
    else:
        break
```


