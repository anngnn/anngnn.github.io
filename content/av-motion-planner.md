---
title: "Autonomous Vehicle Motion Planner"
date: 2026-08-08
draft: false
author: "An Nguyen"
tags:
  - C++
  - Motion Planning
  - Frenet Frame
  - A*
  - OpenCV

image: 
description: ""
toc: true
mathjax: true
socialShare: False
# Repo is private for now. TODO: delete the
# socialShare line above and uncomment repoName below.
# repoName: av-motion-planner
---
## Overview

The goal of this project is to build a motion planning stack for a self-driving car from scratch in C++. The car plans a global route with A\*, generates smooth local trajectories in the Frenet frame, scores them, avoids obstacles, and drives itself to the goal. Everything runs in a 2D OpenCV simulation.

<div style="display: flex; justify-content: center;">
  <img src="/images/projects/av-planner/av-planner-demo.gif" alt="The planner driving the car around obstacles to the goal" style="width: 70%; height: auto;"/>
</div>
<br>

The car follows the A\* route, swerves around obstacles, and stops near the goal.

**Blue** = the car, **green** = the A\* route, **orange** = obstacles, **gray fan** = candidate trajectories, **bold red** = the chosen collision-free trajectory.

Every layer was written and unit tested by hand without any planning libraries.

---

## Planning Loop

The stack is split into three planning layers: global, local, and behavioral. They run in a loop on every frame.

<div style="max-width: 760px; margin: 0 auto;">
<svg viewBox="0 0 760 300" width="100%" role="img" aria-label="Planning pipeline: A star route and reference line are built once at startup, then a four-stage loop of Frenet candidates, cost and collision, behavior state machine, and control runs every frame." font-family="inherit">
  <defs>
    <marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor" opacity="0.55"/>
    </marker>
  </defs>
  <text x="40" y="26" font-size="12" fill="currentColor" opacity="0.6">once, at startup</text>
  <rect x="40" y="38" width="210" height="54" rx="8" fill="none" stroke="currentColor" stroke-opacity="0.35"/>
  <text x="145" y="60" font-size="13" fill="currentColor" text-anchor="middle">A* route</text>
  <text x="145" y="78" font-size="13" fill="currentColor" text-anchor="middle" opacity="0.7">on the road graph</text>
  <line x1="252" y1="65" x2="292" y2="65" stroke="currentColor" stroke-opacity="0.55" stroke-width="1.5" marker-end="url(#ah)"/>
  <rect x="300" y="38" width="230" height="54" rx="8" fill="none" stroke="currentColor" stroke-opacity="0.35"/>
  <text x="415" y="60" font-size="13" fill="currentColor" text-anchor="middle">reference line</text>
  <text x="415" y="78" font-size="13" fill="currentColor" text-anchor="middle" opacity="0.7">arc length s, heading θ</text>
  <path d="M 415 92 L 415 118 L 110 118 L 110 146" fill="none" stroke="currentColor" stroke-opacity="0.55" stroke-width="1.5" marker-end="url(#ah)"/>
  <rect x="16" y="134" width="728" height="140" rx="12" fill="currentColor" opacity="0.045"/>
  <text x="728" y="150" font-size="12" fill="currentColor" text-anchor="end" opacity="0.6">every frame</text>
  <rect x="30" y="158" width="160" height="58" rx="8" fill="none" stroke="currentColor" stroke-opacity="0.35"/>
  <text x="110" y="182" font-size="13" fill="currentColor" text-anchor="middle">84 Frenet</text>
  <text x="110" y="199" font-size="13" fill="currentColor" text-anchor="middle" opacity="0.7">candidates</text>
  <line x1="192" y1="187" x2="206" y2="187" stroke="currentColor" stroke-opacity="0.55" stroke-width="1.5" marker-end="url(#ah)"/>
  <rect x="210" y="158" width="160" height="58" rx="8" fill="none" stroke="currentColor" stroke-opacity="0.35"/>
  <text x="290" y="182" font-size="13" fill="currentColor" text-anchor="middle">cost +</text>
  <text x="290" y="199" font-size="13" fill="currentColor" text-anchor="middle" opacity="0.7">collision check</text>
  <line x1="372" y1="187" x2="386" y2="187" stroke="currentColor" stroke-opacity="0.55" stroke-width="1.5" marker-end="url(#ah)"/>
  <rect x="390" y="158" width="160" height="58" rx="8" fill="none" stroke="currentColor" stroke-opacity="0.35"/>
  <text x="470" y="182" font-size="13" fill="currentColor" text-anchor="middle">behavior FSM</text>
  <text x="470" y="199" font-size="13" fill="currentColor" text-anchor="middle" opacity="0.7">CRUISE / STOP</text>
  <line x1="552" y1="187" x2="566" y2="187" stroke="currentColor" stroke-opacity="0.55" stroke-width="1.5" marker-end="url(#ah)"/>
  <rect x="570" y="158" width="160" height="58" rx="8" fill="none" stroke="currentColor" stroke-opacity="0.35"/>
  <text x="650" y="182" font-size="13" fill="currentColor" text-anchor="middle">pure pursuit</text>
  <text x="650" y="199" font-size="13" fill="currentColor" text-anchor="middle" opacity="0.7">+ speed control</text>
  <path d="M 650 218 L 650 250 L 110 250 L 110 220" fill="none" stroke="currentColor" stroke-opacity="0.55" stroke-width="1.5" stroke-dasharray="5 4" marker-end="url(#ah)"/>
  <text x="380" y="270" font-size="12" fill="currentColor" text-anchor="middle" opacity="0.7">replan from the car's new pose</text>
</svg>
</div>

1. **Global route:** A\* searches a road graph for a path from start to goal.
2. **Reference line:** the jagged A\* path becomes a smooth centerline with arc length and heading at every point.
3. **Candidate generation:** trajectories are sampled in the Frenet frame by sweeping lateral offset, time horizon, and target speed.
4. **Scoring:** every candidate gets a cost from jerk, lane center deviation, speed error, and obstacle proximity. The cheapest collision free one wins.
5. **Behavior:** a feasibility based state machine. CRUISE when a safe path exists, STOP when the road is blocked or the goal is reached.
6. **Control:** pure pursuit steering and proportional speed control execute the chosen trajectory on a kinematic bicycle model.

The planner then replans from the car's new state.

The code is divided into several layers:

| Layer | Responsibility | Code |
|---|---|---|
| **Vehicle** | Kinematic bicycle model | `vehicle/kinematic_model.*` |
| **Global** | Road graph and A\* search | `planning/road_graph.*`, `astar.*` |
| **Local** | Frenet transforms, polynomials, cost | `planning/frenet.*`, `*_polynomial.*` |
| **Behavioral** | Feasibility based state machine | `planning/behavior.hpp` |
| **Simulation** | Render loop, controller, obstacles | `src/main.cpp` |

---

## Global Route

The road network is a graph of nodes (intersections) and bidirectional edges (road segments). A\* searches it once at startup and uses straight line distance to the goal as the heuristic:

$$
f(n) = g(n) + h(n)
$$

where:
- \( f(n) \) : estimated total cost of a route passing through node \( n \)
- \( g(n) \) : cost already accumulated to reach \( n \)
- \( h(n) \) : Euclidean distance from \( n \) to the goal

The heuristic never overestimates the remaining cost, so A\* returns the shortest route. The result is a list of waypoints. It describes which way to go, but it is too jagged for a car to drive.

---

## The Frenet Frame

The local planner works in road relative coordinates \( (s, d) \) instead of world coordinates \( (x, y) \):

- \( s \) : how far along the road the car is (arc length)
- \( d \) : how far to the side of the centerline the car is (signed lateral offset)

<div style="max-width: 760px; margin: 0 auto;">
<svg viewBox="0 0 760 262" width="100%" role="img" aria-label="The Frenet frame: a curved reference line with arc length s measured along it, lateral offset d measured perpendicular to it, the car offset from the line, and a candidate trajectory swerving around an obstacle." font-family="inherit">
  <defs>
    <marker id="ah2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor" opacity="0.6"/>
    </marker>
  </defs>
  <path d="M 350 135 C 470 135, 520 90, 660 88" fill="none" stroke="currentColor" stroke-opacity="0.3" stroke-width="3"/>
  <path d="M 70 215 C 190 215, 230 135, 350 135" fill="none" stroke="#2f9e6e" stroke-width="3.5"/>
  <circle cx="497" cy="112" r="18" fill="#f08c00" opacity="0.9"/>
  <path d="M 350 80 C 400 45, 460 40, 520 60 C 570 76, 600 86, 645 88" fill="none" stroke="#d94141" stroke-width="2.5" stroke-dasharray="7 5"/>
  <line x1="350" y1="135" x2="350" y2="88" stroke="currentColor" stroke-opacity="0.75" stroke-width="1.5" stroke-dasharray="4 4"/>
  <circle cx="350" cy="135" r="4.5" fill="none" stroke="currentColor" stroke-opacity="0.75" stroke-width="1.5"/>
  <line x1="358" y1="135" x2="428" y2="134" stroke="currentColor" stroke-opacity="0.6" stroke-width="1.5" marker-end="url(#ah2)"/>
  <circle cx="350" cy="80" r="8" fill="#2f6bd8"/>
  <line x1="350" y1="80" x2="372" y2="66" stroke="#d94141" stroke-width="2"/>
  <circle cx="70" cy="215" r="4" fill="#2f9e6e"/>
  <text x="70" y="240" font-size="13" fill="currentColor" text-anchor="middle" opacity="0.8">s = 0</text>
  <text x="205" y="199" font-size="14" fill="#2f9e6e" text-anchor="middle" font-style="italic">s</text>
  <text x="340" y="112" font-size="14" fill="currentColor" text-anchor="end" font-style="italic">d</text>
  <text x="330" y="76" font-size="13" fill="currentColor" text-anchor="end" opacity="0.8">car</text>
  <text x="436" y="140" font-size="14" fill="currentColor" opacity="0.8" font-style="italic">θ</text>
  <text x="497" y="150" font-size="13" fill="currentColor" text-anchor="middle" opacity="0.8">obstacle</text>
  <text x="445" y="32" font-size="13" fill="#d94141" text-anchor="middle">candidate</text>
  <text x="660" y="118" font-size="13" fill="currentColor" text-anchor="end" opacity="0.65">reference line</text>
</svg>
</div>

*The reference line is the smoothed A\* route. A position is described by how far along it the car is (\( s \), green) and how far to the side it sits (\( d \), dashed), where \( \theta \) is the road heading at that point.*

This turns a 2D curvy road problem into two 1D problems: make progress along the road, and pick a lane offset. A swerve becomes a single choice of \( d \) over time.

The A\* waypoints are first converted into a reference line, a sequence of anchor points that each store a world position, the path tangent \( \theta \), and the cumulative arc length \( s \).

To convert a world point into Frenet coordinates, the planner finds the nearest reference point and projects the offset vector onto the road frame:

$$
s = s_{i} + \Delta \cdot \begin{bmatrix} \cos\theta \\ \sin\theta \end{bmatrix}, \qquad
d = \Delta \cdot \begin{bmatrix} -\sin\theta \\ \cos\theta \end{bmatrix}
$$

where:
- \( i \) : index of the reference point nearest the query point
- \( \Delta = (x - x_i,\ y - y_i) \) : vector from that reference point to the query point
- \( s_i \) : arc length stored at that reference point
- \( \theta \) : road heading at that reference point

The along track projection extends \( s \) past the anchor. The cross track projection gives the signed lateral offset, with \( +d \) to the left of the direction of travel.

The inverse transform interpolates a base point at arc length \( s \) between the two bracketing reference points, then steps \( d \) meters perpendicular to the interpolated heading:

$$
p_\mathrm{world} = p_\mathrm{ref}(s) + d \begin{bmatrix} -\sin\theta(s) \\ \cos\theta(s) \end{bmatrix}
$$

where:
- \( p_\mathrm{world} \) : the resulting world position
- \( p_\mathrm{ref}(s) \) : point on the reference line at arc length \( s \), interpolated between the two bracketing anchors
- \( \theta(s) \) : road heading at that same point

Trajectories often extend past the end of the reference line, so \( s \) is clamped to the range of the line before interpolating. Without the clamp there is no bracketing pair, the interpolation divides by zero, and the resulting NaN sends the car off the canvas.

---

## Generating Candidate Trajectories

With the road flattened into \( (s, d) \), a trajectory is two independent functions of time. Both are minimum jerk so the motion stays smooth instead of twitchy.

**Lateral Motion**  
A quintic polynomial is used because the lateral maneuver is fully constrained. The car starts at some offset and must arrive at a target offset with no lateral velocity or acceleration:

$$
d(t) = a_0 + a_1 t + a_2 t^2 + a_3 t^3 + a_4 t^4 + a_5 t^5
$$

where:
- \( t \) : time since the start of the maneuver
- \( T \) : the time horizon of the maneuver, meaning how long it lasts
- \( a_0 \) to \( a_5 \) : coefficients solved from the boundary conditions

Six boundary conditions (position, velocity, and acceleration at \( t = 0 \) and \( t = T \)) give six coefficients. The first three come directly from the start state. The remaining three are solved as a 3x3 linear system with Eigen.

**Longitudinal Motion**  
A quartic polynomial is used because the car only needs to reach a cruising speed. Where it ends up along the road is left free:

$$
s(t) = a_0 + a_1 t + a_2 t^2 + a_3 t^3 + a_4 t^4
$$

Here \( a_0 \) to \( a_4 \) are the five coefficients of the longitudinal polynomial, and \( t \) and \( T \) mean the same as above. Dropping the end position constraint removes one equation, so the longitudinal polynomial is one degree lower than the lateral one.

The generator sweeps three axes to build the fan of candidates:

- **Lateral offset \( d \):** ±3.0 m from the centerline, 7 samples
- **Time horizon \( T \):** 2.0 to 5.0 s, 4 samples
- **Target speed:** 5.0 ± 1.0 m/s, 3 samples

That gives 84 candidates per frame. Each one is sampled every 0.1 s and converted back to world coordinates for drawing and collision checking. This is the gray fan in the visualization.

<div style="max-width: 760px; margin: 0 auto;">
<svg viewBox="0 0 760 440" width="100%" role="img" aria-label="Top: a fan of candidate trajectories leaving the car and ending at different lateral offsets within the road width. Bottom left: quintic lateral curves each ending at a fixed offset. Bottom right: quartic longitudinal curves each settling at a target speed with no fixed end position." font-family="inherit">
  <text x="390" y="26" font-size="13" fill="currentColor" text-anchor="middle" opacity="0.7">7 lateral offsets × 4 horizons × 3 speeds = 84 candidates</text>
  <line x1="60" y1="108" x2="720" y2="108" stroke="currentColor" stroke-opacity="0.25" stroke-width="1.2" stroke-dasharray="6 5"/>
  <line x1="60" y1="192" x2="720" y2="192" stroke="currentColor" stroke-opacity="0.25" stroke-width="1.2" stroke-dasharray="6 5"/>
  <line x1="60" y1="150" x2="720" y2="150" stroke="currentColor" stroke-opacity="0.35" stroke-width="2.5"/>
  <path d="M 90 150 C 180 150, 300 122, 400 122" fill="none" stroke="#5b7fa6" stroke-width="1.6" opacity="0.4"/>
  <path d="M 90 150 C 320 150, 520 122, 700 122" fill="none" stroke="#5b7fa6" stroke-width="1.6" opacity="0.4"/>
  <path d="M 90 150 C 260 150, 400 192, 560 192" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <path d="M 90 150 C 260 150, 400 178, 560 178" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <path d="M 90 150 C 260 150, 400 164, 560 164" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <path d="M 90 150 C 260 150, 400 150, 560 150" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <path d="M 90 150 C 260 150, 400 136, 560 136" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <path d="M 90 150 C 260 150, 400 122, 560 122" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <path d="M 90 150 C 260 150, 400 108, 560 108" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <circle cx="90" cy="150" r="7" fill="#2f6bd8"/>
  <text x="90" y="130" font-size="12" fill="currentColor" text-anchor="middle" opacity="0.8">car</text>
  <text x="716" y="103" font-size="11" fill="currentColor" text-anchor="end" opacity="0.55">+3 m</text>
  <text x="716" y="205" font-size="11" fill="currentColor" text-anchor="end" opacity="0.55">−3 m</text>
  <text x="716" y="166" font-size="11" fill="currentColor" text-anchor="end" opacity="0.55">reference line</text>
  <text x="215" y="256" font-size="13" fill="currentColor" text-anchor="middle">lateral offset d(t): quintic</text>
  <line x1="80" y1="272" x2="80" y2="400" stroke="currentColor" stroke-opacity="0.4" stroke-width="1.2"/>
  <line x1="80" y1="336" x2="350" y2="336" stroke="currentColor" stroke-opacity="0.2" stroke-width="1.2" stroke-dasharray="4 4"/>
  <path d="M 80 336 C 170 336, 250 276, 330 276" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <path d="M 80 336 C 170 336, 250 306, 330 306" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <path d="M 80 336 C 170 336, 250 336, 330 336" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <path d="M 80 336 C 170 336, 250 366, 330 366" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <path d="M 80 336 C 170 336, 250 396, 330 396" fill="none" stroke="#5b7fa6" stroke-width="1.8"/>
  <text x="72" y="278" font-size="11" fill="currentColor" text-anchor="end" opacity="0.6">d</text>
  <text x="350" y="418" font-size="11" fill="currentColor" text-anchor="end" opacity="0.6">t → T</text>
  <text x="215" y="434" font-size="11" fill="currentColor" text-anchor="middle" opacity="0.65">each ends at a chosen offset, laterally at rest</text>
  <text x="565" y="256" font-size="13" fill="currentColor" text-anchor="middle">longitudinal speed: quartic</text>
  <line x1="440" y1="272" x2="440" y2="400" stroke="currentColor" stroke-opacity="0.4" stroke-width="1.2"/>
  <line x1="440" y1="400" x2="710" y2="400" stroke="currentColor" stroke-opacity="0.4" stroke-width="1.2"/>
  <line x1="440" y1="300" x2="710" y2="300" stroke="currentColor" stroke-opacity="0.2" stroke-width="1.2" stroke-dasharray="4 4"/>
  <path d="M 440 372 C 530 372, 610 282, 690 282" fill="none" stroke="#2f9e6e" stroke-width="1.8"/>
  <path d="M 440 372 C 530 372, 610 300, 690 300" fill="none" stroke="#2f9e6e" stroke-width="1.8"/>
  <path d="M 440 372 C 530 372, 610 318, 690 318" fill="none" stroke="#2f9e6e" stroke-width="1.8"/>
  <text x="716" y="304" font-size="11" fill="currentColor" text-anchor="end" opacity="0.6">5 m/s</text>
  <text x="432" y="278" font-size="11" fill="currentColor" text-anchor="end" opacity="0.6">speed</text>
  <text x="710" y="418" font-size="11" fill="currentColor" text-anchor="end" opacity="0.6">t → T</text>
  <text x="565" y="434" font-size="11" fill="currentColor" text-anchor="middle" opacity="0.65">each settles at a target speed; end position left free</text>
</svg>
</div>

*Top: the fan produced from a single start state, bounded by the ±3 m road width. Bottom: the lateral curves must arrive at a chosen offset (six constraints, quintic), while the longitudinal curves only have to reach a speed and end up wherever they end up (five constraints, quartic).*

---

## Scoring

Every candidate gets a scalar cost and the cheapest collision free one wins:

$$
\begin{aligned}
J = \ & w_j \sum_k \Big[ \big( \dddot{s}_k \big)^2 + \big( \dddot{d}_k \big)^2 \Big] \\
      & + w_d\, d(T)^2 \\
      & + w_v \big( \dot{s}(T) - v_\mathrm{target} \big)^2 \\
      & + w_o \sum_k \sum_i \frac{1}{\max(\rho_{ki},\ \epsilon)}
\end{aligned}
$$

where:
- \( J \) : total cost of one candidate, lower is better
- \( k \) : index of a point along the trajectory, sampled every 0.1 s
- \( i \) : index of an obstacle
- \( T \) : the time horizon of the trajectory, so \( d(T) \) and \( \dot{s}(T) \) are values at the end of the maneuver
- \( \dddot{s}_k,\ \dddot{d}_k \) : jerk along the road and across the road at point \( k \)
- \( v_\mathrm{target} \) : desired cruising speed (5 m/s)
- \( \rho_{ki} \) : distance from point \( k \) to the edge of obstacle \( i \), which is the center distance minus the obstacle radius
- \( \epsilon \) : a floor of 0.1 m so \( 1/\rho \) stays finite at the edge
- \( w_j, w_d, w_v, w_o \) : the four weights, set to 1, 1, 1, and 5

Dots are time derivatives. One dot is speed, two dots is acceleration, and three dots is jerk, which is how abruptly the acceleration changes.

Each term encodes one preference:

- **Jerk:** penalizes jolts on both axes, so comfortable trajectories win
- **Off center:** pulls the car back to the lane center once it no longer needs to be elsewhere
- **Speed error:** keeps the car near the target cruising speed
- **Obstacle proximity:** inverse distance to each obstacle, so getting close is expensive even when it is not a collision

Squaring is used in the first three terms so that large errors count more than small ones and the sign does not matter. A drift of one meter left is penalized the same as one meter right.

The proximity term is what makes the avoidance look natural. A pure collision check is binary, so the car has no reason to move until a candidate actually hits an obstacle, which produces a late and sharp swerve. Inverse distance makes nearly hitting something expensive too, so the car starts drifting wide early. I set \( w_o \) to 5 times the other weights to prioritize early avoidance.

Collision checking inflates every obstacle by the radius of the car so each trajectory can be tested as a series of points. The extra margin also absorbs the tracking error of the pure pursuit controller.

---

## Behavioral State Machine

The driving mode is a two state machine. How the state is decided mattered more than the states themselves.

```
CRUISE: a collision-free trajectory exists and the goal is not reached
STOP:   no collision-free trajectory, or the goal is reached
```

My first version decided the state by distance. A third SLOW state engaged when an obstacle came within a threshold, and STOP engaged when it came closer. This deadlocked immediately. Slowing down near an obstacle keeps the obstacle in range, which keeps the car slow, so a car that spawned near an obstacle froze at the start line and never moved.

The fix was to decide the state by feasibility. The Frenet planner already swerves around anything avoidable, so a nearby obstacle is not a reason to slow down. STOP means there is no way through, which happens when all 84 candidates collide. The SLOW state was removed.

<div style="max-width: 760px; margin: 0 auto;">
<svg viewBox="0 0 760 340" width="100%" role="img" aria-label="Left: with every trajectory point checked, the car sitting inside an obstacle's inflated zone makes all candidates collide, so it freezes. Right: skipping the first few points lets a trajectory that leaves the zone stay valid, so the car escapes." font-family="inherit">
  <text x="200" y="28" font-size="13" fill="currentColor" text-anchor="middle">checking every point</text>
  <circle cx="200" cy="160" r="86" fill="none" stroke="currentColor" stroke-opacity="0.45" stroke-width="1.4" stroke-dasharray="6 5"/>
  <circle cx="200" cy="160" r="34" fill="#f08c00" opacity="0.9"/>
  <text x="200" y="62" font-size="11" fill="currentColor" text-anchor="middle" opacity="0.6">inflated by car_radius</text>
  <path d="M 160 178 C 140 210, 125 240, 115 268" fill="none" stroke="currentColor" stroke-opacity="0.35" stroke-width="1.6" stroke-dasharray="5 4"/>
  <circle cx="151" cy="192" r="4.5" fill="#d94141"/>
  <circle cx="143" cy="206" r="4.5" fill="#d94141"/>
  <circle cx="136" cy="221" r="4.5" fill="#d94141"/>
  <circle cx="129" cy="236" r="4.5" fill="#2f9e6e"/>
  <circle cx="122" cy="252" r="4.5" fill="#2f9e6e"/>
  <circle cx="115" cy="268" r="4.5" fill="#2f9e6e"/>
  <circle cx="160" cy="178" r="7" fill="#2f6bd8"/>
  <text x="284" y="252" font-size="22" fill="#d94141" text-anchor="middle">✗</text>
  <text x="200" y="300" font-size="11.5" fill="currentColor" text-anchor="middle" opacity="0.75">the car's own position collides, so every</text>
  <text x="200" y="318" font-size="11.5" fill="currentColor" text-anchor="middle" opacity="0.75">candidate is rejected, so it is frozen</text>
  <text x="560" y="28" font-size="13" fill="currentColor" text-anchor="middle">skipping the first few</text>
  <circle cx="560" cy="160" r="86" fill="none" stroke="currentColor" stroke-opacity="0.45" stroke-width="1.4" stroke-dasharray="6 5"/>
  <circle cx="560" cy="160" r="34" fill="#f08c00" opacity="0.9"/>
  <path d="M 520 178 C 500 210, 485 240, 475 268" fill="none" stroke="currentColor" stroke-opacity="0.35" stroke-width="1.6" stroke-dasharray="5 4"/>
  <circle cx="511" cy="192" r="4.5" fill="none" stroke="currentColor" stroke-opacity="0.45" stroke-width="1.4"/>
  <circle cx="503" cy="206" r="4.5" fill="none" stroke="currentColor" stroke-opacity="0.45" stroke-width="1.4"/>
  <circle cx="496" cy="221" r="4.5" fill="none" stroke="currentColor" stroke-opacity="0.45" stroke-width="1.4"/>
  <circle cx="489" cy="236" r="4.5" fill="#2f9e6e"/>
  <circle cx="482" cy="252" r="4.5" fill="#2f9e6e"/>
  <circle cx="475" cy="268" r="4.5" fill="#2f9e6e"/>
  <circle cx="520" cy="178" r="7" fill="#2f6bd8"/>
  <text x="644" y="252" font-size="22" fill="#2f9e6e" text-anchor="middle">✓</text>
  <text x="560" y="300" font-size="11.5" fill="currentColor" text-anchor="middle" opacity="0.75">the first points are skipped, so a path that</text>
  <text x="560" y="318" font-size="11.5" fill="currentColor" text-anchor="middle" opacity="0.75">leaves the zone stays valid, so it escapes</text>
</svg>
</div>

*Hollow dots are the skipped points and filled green dots are checked. The car cannot un-occupy where it already is, so judging those first points rejects every candidate at once.*

A second deadlock came from the collision check itself. If the car drifted inside the inflated zone of an obstacle, every candidate started inside that zone and was flagged as a collision, so the car froze with nothing to follow. The collision check now skips the first few points of each trajectory, since only where a trajectory is going should count. A path that exits the zone stays valid and the car can drive back out.

---

## Control

The chosen trajectory is executed on a kinematic bicycle model. It enforces non-holonomic motion, so the car cannot slide sideways and has a minimum turning radius the planner has to respect:

$$
\dot{x} = v\cos\theta, \qquad
\dot{y} = v\sin\theta, \qquad
\dot{\theta} = \frac{v}{L}\tan\delta
$$

where:
- \( v \) : the car's speed
- \( L \) : the wheelbase
- \( \delta \) : the steering angle
- \( \theta \) : the heading of the car in the world frame, not the road heading from the Frenet section
- \( \dot{x},\ \dot{y} \) : how fast the car moves along each world axis
- \( \dot{\theta} \) : how fast the car turns

**Steering**  
Pure pursuit aims at a point 12 samples ahead on the freshly planned trajectory and steers by the heading error to that point, wrapped to \( [-\pi, \pi] \) so the car turns the short way. The trajectory is replanned every frame, so there is no need to track progress along the old plan.

<div style="max-width: 760px; margin: 0 auto;">
<svg viewBox="0 0 760 268" width="100%" role="img" aria-label="Pure pursuit: the car aims at a point twelve samples ahead on the freshly planned trajectory, and the angle between its current heading and the direction to that point becomes the steering command." font-family="inherit">
  <defs>
    <marker id="ah5" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor" opacity="0.6"/>
    </marker>
  </defs>
  <path d="M 110 200 L 163 216 A 55 55 0 0 0 163 184 Z" fill="#d94141" opacity="0.13"/>
  <path d="M 110 200 C 260 205, 330 100, 620 60" fill="none" stroke="#d94141" stroke-width="2.5" stroke-dasharray="7 5"/>
  <line x1="110" y1="200" x2="312" y2="147" stroke="currentColor" stroke-opacity="0.5" stroke-width="1.4" stroke-dasharray="2 4"/>
  <line x1="110" y1="200" x2="252" y2="241" stroke="currentColor" stroke-opacity="0.7" stroke-width="1.8" marker-end="url(#ah5)"/>
  <path d="M 163 216 A 55 55 0 0 0 163 184" fill="none" stroke="#d94141" stroke-width="1.8"/>
  <circle cx="126" cy="199" r="3" fill="currentColor" opacity="0.5"/>
  <circle cx="142" cy="198" r="3" fill="currentColor" opacity="0.5"/>
  <circle cx="158" cy="196" r="3" fill="currentColor" opacity="0.5"/>
  <circle cx="174" cy="193" r="3" fill="currentColor" opacity="0.5"/>
  <circle cx="193" cy="190" r="3" fill="currentColor" opacity="0.5"/>
  <circle cx="212" cy="186" r="3" fill="currentColor" opacity="0.5"/>
  <circle cx="231" cy="180" r="3" fill="currentColor" opacity="0.5"/>
  <circle cx="250" cy="174" r="3" fill="currentColor" opacity="0.5"/>
  <circle cx="270" cy="167" r="3" fill="currentColor" opacity="0.5"/>
  <circle cx="291" cy="157" r="3" fill="currentColor" opacity="0.5"/>
  <circle cx="312" cy="147" r="6.5" fill="none" stroke="currentColor" stroke-width="2"/>
  <circle cx="110" cy="200" r="8" fill="#2f6bd8"/>
  <text x="94" y="206" font-size="12" fill="currentColor" text-anchor="end" opacity="0.8">car</text>
  <text x="184" y="206" font-size="12" fill="#d94141">steering</text>
  <text x="258" y="248" font-size="12" fill="currentColor" opacity="0.75">current heading θ</text>
  <text x="214" y="152" font-size="11" fill="currentColor" text-anchor="end" opacity="0.65">aim straight at it</text>
  <text x="330" y="172" font-size="12" fill="currentColor" opacity="0.85">lookahead point</text>
  <text x="330" y="190" font-size="11" fill="currentColor" opacity="0.6">12 samples along the fresh plan</text>
  <text x="616" y="56" font-size="12" fill="#d94141" text-anchor="end">chosen trajectory</text>
</svg>
</div>

*The steering command is the angle between where the car is pointing and where the lookahead point sits.*

**Speed**  
A proportional controller sets the commanded acceleration \( a = K_p (v_\mathrm{desired} - v) \), where \( K_p \) is the gain. This is what makes the state machine reach the wheels. CRUISE targets the cruising speed and STOP targets zero.

---

## Testing

Unit tests are written with Catch2 and cover the layers where a math error would be hard to spot from the visualization alone:

- **Road graph and A\*:** connectivity, shortest route correctness, and the case with no path.
- **Frenet transforms:** arc length and heading along the reference line, signed lateral offset, and world to Frenet to world round trips.
- **Quintic and quartic polynomials:** boundary conditions at \( t = 0 \) and \( t = T \), and matching derivatives.
- **Cost and collision:** each penalty term responds in the right direction, and collision detection respects the inflated radius.

---

## Tools

- C++23
- CMake
- Eigen (solving for the polynomial coefficients)
- OpenCV (2D visualization and render loop)
- Catch2 v3 (unit tests)
