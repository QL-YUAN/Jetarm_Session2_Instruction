# Pick and Place Programming Instruction

> **SIT Restricted**

---

## Objective

The objective of this document is to introduce learners to completing a pick-and-place programming implementation based on existing program function packages.

The covered content includes:
- Necessary frame poses for camera and end-effector gripper  
- Tag tracking pose representation  
- Pick-and-place programming instructions  

---

## Prerequisites

- Basic understanding of ROS Python programming  
- Familiarity with matrices, vectors, and trigonometry  
- Basic knowledge of robotic kinematics, trajectory planning, and control concepts  

---

## Equipment

- JetArm Robot with Orin Nano Board  
- Ubuntu environment with ROS and JetArm ROS packages  
- MATLAB  

---

## Lab Setup

Ensure the JetArm and calibrated camera are working properly.

---

## 1. Necessary Frame Poses

### 1.1 Pose of Camera Frame in `link5` Frame


$$
{}^{\text{link5}}T_{\text{camera}} = translation([-0.045, 0, 0.02]) · rotz(-90°)
$$
---

$$
{}^{\text{link5}}T_{\text{camera}} =
\begin{bmatrix}
0 & 1 & 0 & -0.045 \\
-1 & 0 & 0 & 0 \\
0 & 0 & 1 & 0.02 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

- Position (x, y, z): `[-0.045, 0, 0.02]`  
- Orientation (yaw, pitch, roll): `[-1.57, 0, -0.0]`  

> Minor adjustment may be needed for individual robots.  
> Example: `[-1.57, 0, -0.1]` works for one sample robot.

---

### 1.2 Pose of End-Effector (EEF) Frame in `link5` Frame

```
T = translation([0.0, 0.0, 0.11054687369]) · roty(-90°)
```

- Position (x, y, z): `[0.0, 0.0, 0.11054687369]`  
- Orientation (yaw, pitch, roll): `[0.0, -1.57, 0.0]`  

---
![Camera and End-Effector Frames](camera_eef_frame.png)


**Figure 1.** Camera and End-Effector Frame Poses Illustration

---

## 2. AprilTag Tracking

Using `dt_apriltag` to track object poses, the direct measurement provides the pose of AprilTag markers in the **camera reference frame**.

To determine the pose of the tag in the `base_link` frame, the following steps are required:

1. Compute the forward kinematics to obtain the pose of the end-effector (EEF) in `base_link`.
2. Use the known fixed transformation between the camera frame and the EEF frame.  
   - Refer to `Lab9/matrix_calculation_demo.py` on xSite for reference.
3. Compute the camera pose in `base_link`.
4. Transform the detected tag pose into the `base_link` frame.
5. Use the tag pose for inverse kinematics (IK) to compute joint angles for picking.

### Alternative Forward Kinematics

JetArm provides a ROS service to directly obtain the EEF pose relative to `base_link`:

```
/kinematics/get_current_pose
```

### Practical Note

Due to kinematic chain errors and tag measurement noise, relying on a single measurement may fail.  
It is recommended to:
- Take multiple measurements  
- Average the tag position for improved pose estimation  

---

## 3. Picking Program Instructions

Below are suggested steps if you do not yet have a complete implementation plan.

After detecting the tag pose relative to `base_link`, JetArm IK uses:
```
[x, y, z] + pitch
```
to compute joint angles.
$$
{}^{\text{base}}P_{\text{tag}}=[x, y, z]
$$

Directly using the tag pose may cause collisions with the table. Instead, use an **approach–grasp–retreat** strategy.

---

### Basic Picking Steps

**Step 0 – Initialization**  
- Open the gripper (`gripper = 200`)

**Step 1 – Approaching**  
- Target: `0.1 m` above the tag  
- Pitch: `0`  
- Call IK to figure out joint angles and execute accordingly

**Step 2 – Approaching**  
- Target: `0.05 m` above the tag  
- Pitch: `0`  
- Call IK to figure out joint angles and execute accordingly

**Step 3 – Grasping**  
- Target: tag position  
- Pitch: `0`  
- Close gripper after reaching

**Step 4 – Leaving**  
- Target: `0.1 m` above the tag  
- Pitch: `0`  
- Call IK to figure out joint angles and execute accordingly

**Step 5 – Move to Goal**

---

## Motion Compensation

JetArm’s built-in kinematics define the EEF frame as **11 cm away from `link5`**, which is longer than the actual grasp center.  
This may cause the gripper to miss the object.

To compensate, add motion offsets along the **EEF x-axis (in base frame)**, $${}^{\text{base}}x_{\text{tag}}$$.

### Compensation Strategy (Gripper Flat Pose)

- **Before grasping**: move slightly backward  
  - Example: $$P={}^{\text{base}}P_{\text{tag}}-0.05*{}^{\text{base}}x_{\text{tag}}$$
- **After grasping**: move slightly forward  
  - Example:  $$P={}^{\text{base}}P_{\text{tag}}+0.03*{}^{\text{base}}x_{\text{tag}}$$

---

### Recommended Picking Sequence with Compensation

**Step 0 – Initialization**  
- Open gripper (`gripper = 200`)

**Step 1 – Approaching** 
- set a target 0.1m above the tag, $$P={}^{\text{base}}P_{\text{tag}}+[0,0,0.1]$$, with pitch=0, call IK to figure out joint angles and execute the motion 
- Pitch: `0`

**Step 2 – Approaching**  
- set a target 0.1m above the tag, $$P={}^{\text{base}}P_{\text{tag}}+[0,0,0.5]$$, with pitch=0, call IK to figure out joint angles and execute the motion 
- Pitch: `0`

**Step 3.1 – Grasping (Pre-compensation)**
- set a target -0.05m (along EEF x-axis), $$P={}^{\text{base}}P_{\text{tag}}-0.05*{}^{\text{base}}x_{\text{tag}}$$, with pitch=0, call IK to figure out joint angles and execute the motion 
- Pitch: `0`  


**Step 3.2 – Grasping (Post-compensation)**  
- set a target -0.05m (along EEF x-axis), $$P={}^{\text{base}}P_{\text{tag}}+0.03*{}^{\text{base}}x_{\text{tag}}$$, with pitch=0, call IK to figure out joint angles and execute the motion 
- Pitch: `0`  
- Close gripper

**Step 4 – Leaving**  
- set a target 0.1m above the tag, $$P={}^{\text{base}}P_{\text{tag}}+[0,0,0.1]$$, with pitch=0, call IK to figure out joint angles and execute the motion 
- Pitch: `0`

**Step 5 – Move to Goal**

---

## Placing Operation

Placing follows the same principles as picking.  
Please implement the placing routine independently using the same approach strategy.

---
## Implementation Instructions

### 1. AprilTag Tracking
To learn how to track AprilTags, follow this guide:  
[AprilTag Tag Tracking](Tag_Tracking_Introduction.md)  
You can use the script: `tag_tracking_basic.py` and `tag_tracking_basic_tf.py`

---

### 2. Collecting Images for Calibration
To collect images for camera calibration, follow this guide:  
[Image Collector](Image_Collector_Instruction.md)  
You can use the script: `image_collector.py`

---

### 3. Matrix Calculations for Kinematics and Pose Tracking
For demonstrations on matrix calculations related to robot kinematics and pose tracking, follow this guide:  
[Matrix Calculation Demo](MatrixCalculationDemo.md)  
You can use the script: `matrix_calculation_demo.py`

---

### 4. Getting End-Effector Pose
To directly get the end-effector pose relative to the robot base (`base_link`), follow this guide:  
[Instruction on Get_EEF_Pose](Get_EEF_Pose.md)  
You can use the script: `call_service_sample.py`

---

### 5. Launch File For TF Publication
To publish TF between the robot's EEF (`link5`) and the camera's optical frame:  
[Launch File For TF Publication](launch_file_tf_publication_intro.md)  
You can put following launch file downloaded from Lab9 LMS in launch folder in a jetarm package: `tf_hand2camera.launch`

### 6. Sample of Programing Robot Motion with Motion File: Download from Lab7 in LMS 
* Motion file `matlab_output.txt`
* Program file `execute_joint_angles_import_data_from_file.py`
* This method is not for your pick and place project, just an example on how to set waypoints in a file for robot execution. 
