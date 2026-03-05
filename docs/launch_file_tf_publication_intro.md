# EEF to Camera Static Transform Launch

## Purpose

This ROS launch file defines a **static transform** between the robot's **end-effector (EEF)** and an **RGB-D camera** mounted on the robot.  

By publishing this static transform, the system allows ROS nodes to know the **fixed spatial relationship** between the robot's EEF (`link5`) and the camera's optical frame (`rgbd_cam_color_optical_frame`). This is essential for tasks like:

- Visualize the frame in TF in rviz
- Camera-based manipulation
- Visual tracking of objects relative to the robot
- Kinematics calculations involving both the arm and the camera

## Launch File Details

```xml
<?xml version="1.0" encoding="UTF-8"?>
<launch>  
  <node pkg="tf" type="static_transform_publisher" name="eef_to_camera_transform" 
        args="-0.045 0.0 0.02 -1.57 0 0 link5 rgbd_cam_color_optical_frame 100"/>
</launch>

