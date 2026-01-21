---
title: "Gripper and Droxel Design for Drone Construction"
date: 2025-10-01
draft: false
author: "An Nguyen"
tags:
  - CAD
  - Mechanical Design
  - Passive Grippers


image: 
description: ""
toc: true
mathjax: true
socialShare: False

---
## Overview
The goal of this project is to design and validate a lightweight, passive gripper and droxels that enable drones to reliably pick up, transport, and place modular structures under tight weight and control constraints.

---
## Gripper Design
Lightweight passive grippers are preferred due to the drones’ weight and control constraints. Each gripper must be capable of attaching to, carrying, and releasing the building blocks reliably.

**Previous designs**:

<div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/droxel/gripperv1.gif" alt="gripperv1" style="width: 48%; height: auto;"/>
    <img src="/images/projects/droxel/gripperv2.gif" alt="gripperv2" style="width: 48%; height: auto;"/>
</div>
<br>

The first version requires the drone to apply a substantial amount of force to pick up the droxel. It also needs to exert the same amount of force and ascend quickly to detach from it.

The second version is too bulky, and even when operated manually, a person must be very careful when sliding in the droxel. Otherwise, it tends to rotate diagonally and get stuck in the frame, preventing the gripper from releasing it.

**Gripper - Third Version**:
<div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/droxel/gripperv3.gif" alt="gripperv3" style="width: 48%; height: auto;"/>
    <img src="/images/projects/droxel/drox_attached.jpg" alt="a droxel attached to a drone" style="width: 48%; height: auto;"/>
</div>
<br>

The third design uses two magnets, one embedded on top of the droxel and the other placed deeper inside the gripper. A small gap between the top of the gripper and its internal magnet reduces the magnetic force, allowing the droxel to detach easily when the drone twists in the yaw direction.

**Gripper - Final Version**:
<div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/droxel/conegripper.gif" alt="gripperv3" style="width: 48%; height: auto;"/>
</div>
<br>

The final design makes use of a conical shape to maximize pickup tolerance when the gripper is close to the four small towers of the droxel. It passively self-corrects as the gripper descends, guiding itself into alignment when it contacts one of the towers.

---
## Droxel Design
The droxel needs to be lightweight enough for the drones to pick up, carry for an extended period without dipping, and place accurately at the desired destination.

The maximum allowable weight for the droxel—so the drone does not dip during flight—was determined to be under 18 g. From this budget, 1 g was reserved for the embedded magnetic cube.

Due to the geometric structure of the droxel, increased width makes pickup easier. However, weight constraints force the droxel to become very short if the design prioritizes width too heavily.

**Droxel - Wide**:
<div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/droxel/droxel_CAD/drox-v1.png" alt="droxv4" style="width: 48%; height: auto;"/>
</div>
<br>


A taller droxel improves placement accuracy, as its side walls provide more guidance and greater tolerance for positional error during placement. As a result, a balance between width and height was established through multiple design iterations.

**Droxel - Tall**:
<div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/droxel/droxel_CAD/drox-v2.png" alt="droxv4" style="width: 48%; height: auto;"/>
</div>
<br>

A wider top also allows the conical gripper to be guided more easily toward the embedded magnet.

**Droxel - Wide with More Space Between Towers**:
<div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/droxel/droxel_CAD/drox-v4.png" alt="droxv4" style="width: 48%; height: auto;"/>
</div>
<br>

The finalized version smooths the bases of the four towers to allow the conical gripper to slide down easily.

**Droxel - Final Version**:
<div style="display: flex; justify-content: space-between;">
    <img src="/images/projects/droxel/droxel_CAD/drox-vfinal.png" alt="droxvfinal" style="width: 48%; height: auto;"/>
    <img src="/images/projects/droxel/droxel_CAD/drox-gripper-vfinal.png" alt="droxvfinal" style="width: 48%; height: auto;"/>
</div>
<br>

---
## Pick and Place
<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/drone/pickplace_block3x.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

---
## Acknowlegements
I would like to thank Professor Latteur for sharing the droxel Grasshopper file, which made it possible for me to customize and adapt the droxel parameters for this project.

---
## Research
[1] P. Latteur, A. Fascetti, and S. Goessens, “The droxel: A universal parametric construction component,” *Engineering Structures*, vol. 250, p. 113363, 2022.
