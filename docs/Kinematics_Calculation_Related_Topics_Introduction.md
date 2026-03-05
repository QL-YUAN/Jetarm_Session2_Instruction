## Kinematics Related Topics Instruction

### 1. Forward Kinematics
[JetArm Forward Kinematics Introduction](JetArm_ForwardKinematics_Intro.md)  
You can use the script: `fk.py` and `fk_exercise.py`

---

### 2. Inverse Kinematics
[Inverse Kinematics Introduction](JetArm_IK_Intro.md)  
You can use the script: `ik_english.py`

---

### 3. Robot Kinematics Parameters
- Check the details in script: `jetarm_6dof_params.py.py`
- This could be different from the DH parameter you derived in class. 

```
'''
Modified DH: 
Attention: This is not the same DH parameter presentation for the assignment.
----------------------------------------------
i | α(i-1) | a(i-1) |       θ(i)      | d(i) |
----------------------------------------------
1 |   0°   |   0    |  θ1(-120, 120)  |   0  |
----------------------------------------------
2 |  -90°  |   0    |  θ2(-180, 0)    |   0  |
----------------------------------------------
3 |   0°   | link1  |  θ3(-120, 120)  |   0  |
----------------------------------------------
4 |   0°   | link2  |  θ4(-200, 20)   |   0  |
----------------------------------------------
5 |  -90°  |   0    |  θ5(-120, 120)  |   0  |
----------------------------------------------
'''

# Link length (m)
# The height of the base, here the origin of the first coordinate system is aligned with the origin of the second coordinate system
base_link = 0.10314916202

link1 = 0.12941763737
link2 = 0.12941763737

# When calculating the tool_link, it takes the value of link3 + tool_link, because the origin of the end effector's coordinate system coincides with the previous one
# Here, tool_link refers to the actual gripper length
link3 = 0.05945583202
tool_link = 0.112

# Joint angle limits, depending on collision and the rotation range of the servo
# An additional 0.2 is added to prevent numerical instability during calculation, making it slightly larger than the set value
joint1 = [-120.2, 120.2]
joint2 = [-180.2, 0.2]
joint3 = [-120.2, 120.2]
joint4 = [-200.2, 20.2]
joint5 = [-120.2, 120.2]

# Servo pulse width range, midpoint value, corresponding angle range, mid-point value
joint1_map = [0, 1000, 500, -120, 120, 0]
joint2_map = [0, 1000, 500, 30, -210, -90]
joint3_map = [0, 1000, 500, 120, -120, 0]
joint4_map = [0, 1000, 500, 30, -210, -90]
joint5_map = [0, 1000, 500, -120, 120, 0]
```

---

### 4. More Kinematics Related Functions 
```
def angle_transform(angle, param, inverse=False):
    if inverse:
        new_angle = ((angle - param[5]) / (param[4] - param[3])) * (param[1] - param[0]) + param[2]
    else:
        new_angle = ((angle - param[2]) / (param[1] - param[0])) * (param[4] - param[3]) + param[5]

    return new_angle

def pulse2angle(pulse):
    theta1 = angle_transform(pulse[0], joint1_map)
    theta2 = angle_transform(pulse[1], joint2_map)
    theta3 = angle_transform(pulse[2], joint3_map)
    theta4 = angle_transform(pulse[3], joint4_map)
    theta5 = angle_transform(pulse[4], joint5_map)
    
    #print(theta1, theta2, theta3, theta4, theta5)
    return radians(theta1), radians(theta2), radians(theta3), radians(theta4), radians(theta5)

def angle2pulse(angle):
    pluse = []
    
    for i in angle:
        theta1 = angle_transform(degrees(i[0]), joint1_map, True)
        theta2 = angle_transform(degrees(i[1]), joint2_map, True)
        theta3 = angle_transform(degrees(i[2]), joint3_map, True)
        theta4 = angle_transform(degrees(i[3]), joint4_map, True)
        theta5 = angle_transform(degrees(i[4]), joint5_map, True)
        
        #print(theta1, theta2, theta3, theta4, theta5)
        pluse.extend([[theta1, theta2, theta3, theta4, theta5]])

    return pluse
```

* Transform pulse list to angles and vice versa:  
* Check details in script: `transform.py`


