---
title: "Glowphonic"
date: 2024-12-11
draft: false
author: "An Nguyen"
tags:
  - C++
  - Arduino
  - Max/MSP
  - Digital Music

image: 
description: ""
toc: true
mathjax: true
repoName: glowphonic
---
## Overview

*Glowphonic* is an interactive digital musical instrument with touch-sensitive petals for playing notes and a 6-DOF IMU that modulates tremolo. It features responsive lighting that blinks to touch or audio amplitude, with sensor data sent to Max/MSP and synchronized visuals controlled by an ESP32.

<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/glowphonic.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

---

## Hardware Overview
Glowphonic integrates multiple sensors and microcontrollers:

- **12-Key Capacitive Touch Sensor**: Plays notes from two keyboards via Max/MSP.
- **6-DOF IMU**: Tilting forward (x-axis) increases the tremolo of an audio sample.
- **Slide Potentiometer**: Controls keyboard volume.
- **Ultrasonic Distance Sensor**: Controls sample loop speed. Closer distances loop faster.

Sensor readings are collected by an Arduino Nano and sent to Max/MSP over serial USB. Max interprets this data and sends visual triggers, such as note activations or amplitude spikes, via a second serial connection to an ESP32-S3. The ESP32-S3 controls a NeoPixel LED strip, blinking in response to these events to create synchronized light feedback.

---

## Design
The electronics are mounted inside a cherry blossom-style lampshade. The touch wires extend to the petals using conductive tape, turning them into large capacitive surfaces. The distance sensor is mounted at the top to remain unobstructed.

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/glowphonic/music-circuit1.png" alt="Circuit Diagram for Sensor Readings" style="width: auto; height: auto;"/>
</div>
<br>

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/glowphonic/music-circuit2.png" alt="Circuit Diagram to Blink LED Lights" style="width: auto; height: auto;"/>
</div>
<br>

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/glowphonic/max-code-annotations.png" alt="Max Code Annotations" style="width: auto; height: auto;"/>
</div>
<br>

---

## Build
<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/glowphonic/glow-interior1.png" alt="Interior" style="width: 48%; height: auto;"/>
  <img src="/images/projects/glowphonic/glow-interior2.png" alt="Full Build" style="width: 48%; height: auto;"/>
</div>
<br>

<div style="display: flex; justify-content: space-between;">
  <img src="/images/projects/glowphonic/glow-build1.png" alt="Glowing" style="width: auto; height: auto;"/>
</div>
<br>

---

## Inspiration
*Glowphonic* was inspired by Jonathan Sparks’ <a href="https://jonathansparks.com/nomis/" target="blank">*NOMIS*</a>, winner of the 2015 Guthman People's Choice Award, which produced melodies and layered glowing loops via gesture and light.

---

## References
- [Tremolo Effect in Max/MSP](https://music.arts.uci.edu/dobrian/maxcookbook/tremolo-effect-sound-file)
- [NeoPixel LED Guide](https://learn.adafruit.com/adafruit-neopixel-uberguide/basic-connections)
- [Ultrasonic Sensor Tutorial](https://www.instructables.com/Simple-Arduino-and-HC-SR04-Example/)
- [MPR121 Capacitive Touch Sensor Guide](https://learn.adafruit.com/adafruit-mpr121-12-key-capacitive-touch-sensor-breakout-tutorial)
- [Music Samples](https://samplefocus.com/)

---

## Hardware Components

- [Arduino Nano](https://store.arduino.cc/products/arduino-nano)
- [ProS3 - ESP32-S3 Development Board](https://www.adafruit.com/product/5401)
- [Ultrasonic Distance Sensor (x4)](https://www.adafruit.com/product/4019)
- [Adafruit 12-Key Capacitive Touch Sensor Breakout](https://www.adafruit.com/product/1982)
- [Adafruit ISM330DHCX 6-DOF IMU](https://www.adafruit.com/product/4502)
- [Slide Potentiometer](https://www.adafruit.com/product/5295)
- [Adafruit NeoPixel Digital RGB LED Strip](https://www.adafruit.com/product/1138)
- [Cherry Blossom Lampshade](https://www.amazon.com/dp/B07MRL3FK2)

**Other materials**:
- USB-C to USB-C cables (x2)
- 5V 5A AC to DC Power Supply for LEDs
- Copper Conductive Tape
- Jumper Wires (50cm)