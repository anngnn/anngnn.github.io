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
socialShare: True
repoName: aerial-painting

---
## Overview
The goal of this project is to develop a control system for drones to paint drawings autonomously. By combining motion capture localization, cascaded PID control, and custom soft brushes, the system achieves the precision needed to create coordinated artwork with multiple drones.

<!-- <video controls autoplay loop muted playsinline width="100%">
  <source src="https://github.com/user-attachments/assets/8dcf3a1d-1328-4966-a1e8-0693f886d256" type="video/mp4">
  Your browser does not support the video tag.
</video> -->

{{<youtube 4PcTPp3bISc>}}

---
## Gripper Design
<div style="display: flex;">
    <img src="/images/projects/drone/yarn-gripper.png" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br>
<!-- Since the drone could only exert a small amount of force, traditional brushes were too hard and could cause the drone to flip over. Hence, a custom soft brush was made with yarn to hold enough paint while being soft enough for the drone to move down against the paper to make the dot painting. Oil paint was chosen for its slow-drying property, and a mix of linseed oil also enhanced the paint's longevity during flight and the amount of paint the drone could hold. -->

Traditional brushes were too stiff and caused the drone to flip over when pressed against the canvas. A custom soft brush made with yarn holds enough paint while remaining soft enough for the drone to make controlled contact. Oil paint mixed with linseed oil was chosen for its slow-drying properties and ability to stay on the brush during flight.


---
## SwarmOS

<div style="display: flex;">
    <img src="/images/projects/drone/swarmOS.png" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br>

The air traffic controller (ATC) sends flight commands to the drones through wifi. The Optitrack system localizes the drones using reflective markers and communicates position data to the ATC through ethernet.

---
## Cascaded PID Control
The initial single-loop position controller lacked the precision needed for accurate painting. A cascaded control system with separate position and velocity PID loops was implemented to improve performance.

<div style="display: flex;">
    <img src="/images/projects/drone/pidloops.png" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br>

- Inner Loop (Velocity) reacts instantly to physical disturbances (like wind or motor changes) before they can affect the position.
- The Outer Loop (Position) then only needs to calculate the desired speed to reach the target.
This architecture provides better damping and faster settling times than a single-loop controller.

---
## Controller Comparison

**Old controller hovering:**
<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/drone/oldcontroller.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

**New controller hovering:**
<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/drone/newcontroller.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>


The old controller caused the drone to drift more while hovering while the new controller provides better stability.

**Old controller hovering performance:**
<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/drone/old-hover/oldsmallxpos.png" alt="old x pos hover" style="width: 48%; height: auto;"/>
  <img src="/images/projects/drone/old-hover/oldsmallypos.png" alt="old y pos hover" style="width: 48%; height: auto;"/>
</div>
<br>

**New controller hovering performance:**
<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/drone/new-hover/smallposx.png" alt="new pos x hover" style="width: 48%; height: auto;"/>
  <img src="/images/projects/drone/new-hover/smallposy.png" alt="new pos y hover" style="width: 48%; height: auto;"/>
</div>
<br>

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/drone/new-hover/smallvelx.png" alt="new pos x hover" style="width: 48%; height: auto;"/>
  <img src="/images/projects/drone/new-hover/smallvely.png" alt="new pos y hover" style="width: 48%; height: auto;"/>
</div>
<br>

<!-- ---
## Velocity and Position Tracking for the New Controller with Diffferent Setpoints

**Tracking x**
<div style="display: flex;">
    <img src="/images/projects/drone/trackingx.png" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br>

**Tracking y**
<div style="display: flex;">
    <img src="/images/projects/drone/trackingy.png" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br> -->

---
## Painting
<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/drone/spedup_star.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

<!-- 
---
## Next Steps
Conduct flight tests with multile drones.

<div style="display: flex;">
    <img src="/images/projects/drone/6drones_hover.gif" alt="gripperv3" style="width: 100%; height: auto;"/>
</div>
<br> -->

---
## Acknowledgements
Special thanks to Drew Curtis, Prof. Matt Elwin, and Prof. Michael Rubenstein for their guidance and support throughout this project.