---
layout: post
title:  "Robotics Exploration Series: Part 1"
date:   2019-01-27 07:05:00 -0700
categories: robotics programming c++ c python algorithms
use_math: true
---

### Introduction 

Robotics has been a fascination of mine for the better part of two decades. I remember reading [Mobile Robots](https://www.amazon.ca/Mobile-Robots-Inspiration-Implementation-Second/dp/1568810970) (written by MIT engineers) and being hooked. Due to lack of experience, lack of tools, lack of money and the (electronically) rather complicated design of their control board, early successes were few and far between. However, it put me on a path to get a strong educational foundation in mechatronics and robotics.

Over the last year, I've been building a small mobile robot platform to serve as a basis for exploring robotics in an applied manner. While I've been exposed to many of the classic robotics algorithms in theory, my intent with this series is to apply them to navigation of a mobile robot around my apartment.

This post introduces the platform and my progress on it so far. Future posts will go into detail on specific robotics algorithms theory & application.

### Chassis & Drivetrain

| Component     | Model/make            | Description:                       | Notes:                                                                             | Price:        | Source:   |
|---------------|-----------------------|------------------------------------|------------------------------------------------------------------------------------|---------------|-----------|------------------------|
| Motors        | Faulhaber 2224U024SR  |                                    | 24V was the only choice on ebay w/ gearhead and encoder.                           | $90/pc.       | [Faulhaber](https://bit.ly/2DKz4iJ)      |  |
| Gearhead      | Faulhaber 23/1 Series | 66:1 Planetary gearhead            |                                                                                    | inc. w/ motor |   [Faulhaber](https://bit.ly/2Wxxx7d)        |
| Encoder       | Faulhaber IE2-16      |                                    | 16 counts / rev is plenty with 66:1 gearheadreduction + drivetrain gear reduction. | inc. w/ motor | [Faulhaber](https://bit.ly/2DLf13U)      | 
| Wheels & Hubs | Polulu                |                                    | Chosen to fit with wheel size.                                                     |               | [Robotshop](https://bit.ly/2gkrH2E) | 
| Gears         | Unknown               | Brass gears of unknown provenance. | Various sizes were bought.                                                         | $5/gear       | Ebay      |
| Axles         | Unknown               | Steel axles of unknown provenance. | Various length/thicknesses were bought.                                            | $5/10 pcs.    | Ebay      | 
| Bearings         | Lynxmotion               |  | 3x8x4mm flanged.                                            | $10/10 pcs.    | [Robotshop](https://bit.ly/2TtTiTn)       |

In order to keep the motion model of the robot as simple as possible, I chose to build a robot that using direct-drive skid-steer drivertrain. For the chassis, I used 8mm black acrylic.

<figure>
  <img src="/assets/robot/robot-side.png" style="width:75%"/> <br>
  <figcaption>Robot viewed from the side</figcaption>
</figure>
<figure>
  <img src="/assets/robot/robot-oblique.png" style="width:75%"/> <br>
  <figcaption>Robot as viewed isometrically</figcaption>
</figure>

Further components visible in the pictures:
- Hokuyo URG-04LX scanning laser rangefinder. 
  - Will be streamed back to a computer & used for mapping and navigation. 
- 4x Maxbotix EZ-1 ultrasonic rangefinders.
  - Will be used for last-ditch onboard obstacle avoidance & probably streamed back to the computer as well.
- Custom circuit board (details below) based on an STM32F401RET6 MCU.
  - The "brain" of the robot. Used to interface with all the onboard sensors & actuators. 
  - Has a Microchip / Roving Networks RN41 Bluetooth transceiver for telemetry & control.
- MPU9250 IMU
  - Streamed back to computer & used in state estimation. 
-  S25FL128 128M flash memory & MB85AS4 4M RAM
  - To be used for buffering & storage of telemetry & scans due to potential bluetooth bandwidth constraints.

The chassis was constructed starting with the drivetrain, as it required the most precise assembly. The figure below shows a side-view cutout. The concentric circles are (from large to small): the brass gears, the bearings and the axles. 

<figure>
  <img src="/assets/robot/gearbox-side.png" style="width:100%"/> <br>
  <figcaption>Side view drawing of gearbox</figcaption>
</figure>

I measured out 4 pieces of acrylic and first cut them to the appropriate size, then drilled the holes for the bearings. Due to the nature of the machine tools I was working with, it took a few tries to get the positioning correct so that the gearbox ran smoothly (the tolerances for misalignment are quite low - on the order of mm's).

As you can see from the pictures, the inside piece of acrylic is taller, allowing the motor, gearbox and drive gear combination to be directly mounted to the acrylic. The end result was two diagonally symmetrical assemblies that needed to combined. To achieve that, I cut a top and bottom plate out of the same 8mm acrylic and clamped them together using bolts. While I experimented with different thicknesses to give the robot more clearance, and also considered different methods of affixing the drivetrain assemblies, this approach provided the most rigidity and ability to de-assemble the robot if necessary.

### Electronics 

To facilitate quick prototyping, the project was started by using an STM32F4 Nucleo board by ST. These boards are cheap, available at many electronics retailers (including [Digikey](https://digikey.com)) and come with a built-in ST-Link in-circuit programmer. Combined with the large amount of broken-out I/O pins make this an ideal starting point.

<figure>
  <img src="/assets/robot/robot-v1-schematic.png" style="width:100%"/> <br>
  <figcaption>Initial schematic of robot control board.</figcaption>
</figure>

<figure>
  <img src="/assets/robot/robot-v1-board-top.png" style="width:100%"/> <br>
  <figcaption>Finished circuit board (top view).</figcaption>
</figure>

<figure>
  <img src="/assets/robot/robot-v1-board-bottom.png" style="width:100%"/> <br>
  <figcaption>Finished circuit board (bottom view).</figcaption>
</figure>








