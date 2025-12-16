---
title: "Clash of Hands"
date: 2024-12-11
draft: false
author: "An Nguyen"
tags:
  - C++
  - Python
  - Computer Vision

image: 
description: ""
toc: true
mathjax: true
socialShare: False

---
## Overview

  We programmed the Allegro Hand to play rock-paper-scissors using machine learning strategies that help it get smarter with every game.

  **Team Member:** Pushkar Dave

<video controls style="width: 100%; height: auto;">
    <source src="/videos/clash-of-hands.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

- **Real-time Gesture Recognition:** Uses MediaPipe to detect rock, paper, and scissors from live webcam input.
- **AI Agents:** Markov Model, Decision Tree
- **Robotic Hand Integration:** Controls an Allegro robotic hand via CAN bus to perform AI moves.
- **Game History Tracking:** Logs game data for ongoing learning and behavior prediction.

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/hand/hands-block-diagram.png" alt="Block Diagram of the Interactions Between Subsystems" style="width: 100%; height: auto;"/>
</div>
<br>

---

## AI Agents
**Markov Transition Matrix**
- Tracks transition probabilities between player moves.
- Predicts the next move based on the last move's transition matrix.

**Decision Tree**
- Uses features like previous move frequency and patterns.
- Trains a decision tree to predict the player’s next move and respond optimally.

---

## Gesture Recognition
The system classifies hand gestures using MediaPipe Hands:
- **Closed Fist** → Rock
- **Open Palm** → Paper
- **Victory Sign** → Scissors

---

## Hardware Components
- Allegro Hand V4
- CAN interface and controller