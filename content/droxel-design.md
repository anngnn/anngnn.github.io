---
title: "Autonomous Aerial Painting with Quadrotors"
date: 2025-10-01
draft: false
author: "An Nguyen"
tags:
  - Python
  - CAD
  - OptiTrack

image: 
description: ""
toc: true
mathjax: true
socialShare: False

---
## Overview
The goal of this project is to develop a control system for a swarm of drones to perform specific tasks.

---
## Gripper Design
Lightweight passive grippers are preferred due to the drones’ weight and control constraints. Each gripper must be capable of attaching to, carrying, and releasing the building blocks reliably.

**Previous designs**:

<div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/construction/gripperv1.gif" alt="gripperv1" style="width: 48%; height: auto;"/>
    <img src="/images/projects/construction/gripperv2.gif" alt="gripperv2" style="width: 48%; height: auto;"/>
</div>
<br>

The first version requires the drone to apply a substantial amount of force to pick up the droxel. It also needs to exert the same amount of force and ascend quickly to detach from it.

The second version is too bulky, and even when operated manually, a person must be very careful when sliding in the droxel. Otherwise, it tends to rotate diagonally and get stuck in the frame, preventing the gripper from releasing it.

**Third design**:
<div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/construction/gripperv3.gif" alt="gripperv3" style="width: 48%; height: auto;"/>
    <img src="/images/projects/construction/drox_attached.jpg" alt="a droxel attached to a drone" style="width: 48%; height: auto;"/>
</div>
<br>

The third design uses two magnets, one embedded on top of the droxel and the other placed deeper inside the gripper. A small gap between the top of the gripper and its internal magnet reduces the magnetic force, allowing the droxel to detach easily when the drone twists in the yaw direction.

**Final design**:
<div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/construction/conegripper.gif" alt="gripperv3" style="width: 48%; height: auto;"/>
</div>
<br>

---
## SwarmOS

<div style="display: flex;">
    <img src="/images/projects/construction/swarmOS.png" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br>

---
## Cascaded PID Control

- Inner Loop (Velocity) reacts instantly to physical disturbances (like wind or motor changes) before they can affect the position.
- The Outer Loop (Position) then only needs to calculate the desired speed to reach the target.
- This faster, inner loop reaction provides inherent damping for the entire system, 
→ better stability and quicker settling times compared to a single loop (slower to react)

<div style="display: flex;">
    <img src="/images/projects/construction/pidloops.png" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br>

---
## Velocity and Position Tracking

#### Tracking x
<div style="display: flex;">
    <img src="/images/projects/construction/trackingx.png" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br>

#### Tracking y
<div style="display: flex;">
    <img src="/images/projects/construction/trackingy.png" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br>

---
## Pick and Place
<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/drone/pickplace_block3x.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

---
## Painting
<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/drone/spedup_star.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

---
## Next Steps
Conduct flight tests with multile drones.

<div style="display: flex;">
    <img src="/images/projects/construction/6drones_hover.gif" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br>



