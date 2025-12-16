---
title: "The Pool-inator"
date: 2024-12-12
draft: false
author: "An Nguyen"
tags:
  - Python
  - ROS 2
  - MoveIt2
  - OpenCV

image: 
description: ""
toc: true
mathjax: true
repoName: poolinator
---
## Overview

The Franka Emika Panda 7-DoF arm plays a game of pool by identifying balls and hitting them into pockets.

**Team Members:** Caroline Terryn, Catherine Maglione, Joseph Blom, Logan Boswell

<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/pool-demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

---

## Project Sequence
<ul>
<li>The game begins with the robot arm moving to the side to avoid obscuring the table.
</li>
<li>The camera detects two AprilTags to determine the transformations of the table, the cue, the end-effector, and the camera, all relative to the base of the Franka.
</li>
<li>The locations of the cue ball (red) and other balls (blue) are updated, and these coordinates are fed into the pool algorithm.
</li>
<li>The robot selects a striking position that enables it to hit the cue ball and pocket another ball.
</li>
<li>After striking, the robot arm moves to the side again to update the new positions of the balls.
</li>
<li>The game continues until all the blue balls are pocketed.
</li>
<li>Then the robot hits the red ball into a pocket to end the game.
</li>
</ul>

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/poolinator/pool.gif" alt="Best Strikes" style="width: 100%; height: auto;"/>
</div>
<br>

---

## Nodes and Python Modules
The project is divided into several subsystems:

<dl>
<dt><strong>AprilTags (Transform Node)</strong></dt>
<dd>
    This node uses the table tag and the cue tag to establish the relationship between various coordinate frames (camera, end-effector, table, etc.) to build a TF tree.
</dd>
<br>

<dt><strong>Computer Vision (Image Processing Node)</strong></dt>
<dd>
    This node processes data from the depth camera and camera info topics to locate the cue ball (red ball) and other balls on the pool table. It integrates OpenCV and ROS2 for image processing and publishes the detected ball positions as TF frames for use in robot control and planning.
</dd>
<br>

<dt><strong>Planning & Execution (Control Node)</strong></dt>
<dd>
    This node manages the robot's movements through various states, such as striking, standby, and home positions.
</dd>
<br>

<dt><strong>Pool Algorithm</strong></dt>
<dd>
    This Python module uses ball coordinates to plan feasible striking positions for the robot.
</dd>
<br>

<dt><strong>World</strong></dt>
<dd>
    This Python module represents the environment, including the table, Franka's platform, and the ceiling-mounted camera. It ensures the robot's path planning avoids obstacles by maintaining spatial relationships and tracking key elements like table corners, pockets, and ball positions using TF transforms.
</dd>
</dl>


<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/poolinator/poolinator-diagram.png" alt="Block Diagram of the Interactions Between Subsystems" style="width: auto; height: auto;"/>
</div>
<br>

---

## Custom ROS 2 MoveIt API
Prior to this project, our team developed a custom MoveIt API to help the Franka arm plan trajectories and control its movements. Key features include:

**Pose-based Planning**  
Plan trajectories to reach specific end-effector poses, such as the strike position or standby position.

**Joint State Planning**  
Plan trajectories to move the robot to predefined joint configurations.

**Cartesian Path Planning**  
Plan linear motions for tasks like moving the end-effector along the x-axis during a cue strike.

**Inverse Kinematics**  
Convert desired end-effector poses into joint configurations for precise execution.

**Planning Scene Setup**  
Create a planning scene with obstacles like the table and camera to ensure collision-free paths.

---

## Computer Vision (Image Processing Node)

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/poolinator/red_ball_rviz_contour.gif" alt="Red Ball Contour" style="width: 48%; height: auto;"/>
  <img src="/images/projects/poolinator/red_ball_rviz_image.gif" alt="Real-time Image Streaming" style="width: 48%; height: auto;"/>
</div>
<br>

The image processing node was developed using an Intel RealSense camera to detect the 3D positions of pool balls, providing positional data to guide the robot in planning strike trajectories.

The image pipeline targeted detection of red (cue ball) and blue (target balls) through HSV-based color segmentation. Contours were filtered by size to remove noise, and the center of mass was computed to determine the x and y positions of each ball.

Depth (z) information was obtained from the RealSense depth stream, with pixel coordinates converted into real-world metric positions using intrinsic parameters from the **camera_info** topic.

Each ball’s position was broadcast as a TF frame relative to the robot’s base frame, enabling control nodes to access and update their live positions during operation.


