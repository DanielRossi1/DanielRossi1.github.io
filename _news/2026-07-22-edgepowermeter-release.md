---
layout: post
date: 2026-07-22
inline: false
related_posts: false
title: "EdgePowerMeter: measuring real edge-AI power draw"
---

:zap: **New Open-Source Tool!**

Released **[EdgePowerMeter](https://github.com/DanielRossi1/EdgePowerMeter)**, an instrument to measure the real power consumption of edge devices _without_ triggering voltage throttling.

It pairs a custom **ESP32-C3 + INA226** acquisition probe with a **PySide6** desktop application, so that inference throughput can be correlated with power draw and reported as **Joules per inference** and **FPS/Watt**, the metrics that actually matter when a model has to run on a battery.

**Repository**: [github.com/DanielRossi1/EdgePowerMeter](https://github.com/DanielRossi1/EdgePowerMeter)
