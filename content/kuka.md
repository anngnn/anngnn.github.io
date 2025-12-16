---
title: "KUKA youBot Mobile Manipulation"
date: 2024-12-11
draft: false
author: "An Nguyen"
tags:
  - Python
  - Trajectory Planning
  - Manipulation
  - Feedforward Control

image: 
description: ""
toc: true
mathjax: true
socialShare: False

---
## Overview

  In this project, I developed a full trajectory planning and control pipeline for the youBot mobile manipulator using Modern Robotics. The robot had to autonomously pick up a block from an initial position and drop it off at a final location in CoppeliaSim, with accurate grasping and movement under a physics engine.

  The system plans and follows a reference trajectory for the end-effector using kinematic simulation and feedback control. The output is a CSV file containing the robot’s full configuration (chassis pose, arm joints, wheel angles, gripper state) over time, which is used to animate and test the robot in CoppeliaSim's physics-based Scene 6.

  <div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/kuka/youBot.gif" alt="kuka youBot" style="width: 100%; height: auto;"/>
  </div>
  <br>

---

## Algorithm Desription
  The system includes three main modules:
  - `NextState`: Simulates the next robot configuration based on the current configuration and commanded speeds using Euler integration.
  - `TrajectoryGenerator`: Builds the reference trajectory for the end-effector in task space, including 8 motion segments with associated gripper states.
  - `FeedbackControl`: Computes the end-effector twist command based on the error between the current and desired configurations using a feedforward plus proportional-integral (PI) controller.

---

## Trajectory Generator
  The `TrajectoryGenerator` creates eight concatenated trajectory segments in SE(3), each starting and ending at rest:
  1. Move to standoff above initial block.
  2. Lower to grasp pose.
  3. Close gripper.
  4. Lift to standoff.
  5. Move to standoff above goal.
  6. Lower to place pose.
  7. Open gripper.
  8. Return to standoff.

  Each segment includes both rotational and translational interpolation, with 0.01s resolution.

---

## Initial Setup
  The initial end-effector pose is purposely offset by:
  - **0.2 m translation** in the X-axis
  - **30° rotation** about the Z-axis

  This ensures a meaningful starting error that the feedback controller must reduce over time.

  The initial configuration vector for the robot is:
  ```
  [chassis φ, x, y, J1-J5, W1-W4, gripper state]
  ```
---

## Feedforward Control
  The computed twist is mapped to wheel and joint velocities using the robot Jacobian:
  - The **base Jacobian** is derived using wheel geometry and screw theory.
  - The **arm Jacobian** is computed via `JacobianBody`.

  The combined Jacobian maps the desired twist to joint and wheel speeds using the pseudo-inverse.

  Velocity saturation ensures the commanded speeds stay within physical actuator limits.

---

## Results
### Best Case
<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/kuka/best.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/kuka/best-error-plot.png" alt="Best Case Error Plot" style="width: 100%; height: auto;"/>
</div>
<br>

### Overshoot Case
<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/kuka/overshoot.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/kuka/overshoot-error-plot.png" alt="Overshoot Case Error Plot" style="width: 100%; height: auto;"/>
</div>
<br>


### New Task
<video controls style="width: 100%; height: auto;">
    <source src="/videos/kuka/newTask.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/kuka/newTask-error-plot.png" alt="New Task Error Plot" style="width: 100%; height: auto;"/>
</div>
<br>