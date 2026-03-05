---
note: 01KJC0EXRXW6H88W233K5G62NR.md
title: 20260212 - 测试intel realsense D405
indexed_at: 2026-03-05T12:25:24.337343+00:00
---

## 摘要
记录 Intel RealSense D405 深度相机的安装和测试过程。测试了 USB 2.1/3.0、不同分辨率（848×480、1280×720）和距离（10cm、15cm）下的测距表现。发现近距离测距盲点增多，USB 3.0 仅提升 FPS 而无效果提升。

## 关键概念
- Intel RealSense D405: 近距离深度相机型号，适用于短距离测距场景
- USB 2.1/3.0: 不同 USB 版本影响 FPS，但对测距效果无明显提升
- 测距盲区: 距离越近（如 10cm），不能测距的像素点越多
- librealsense: Intel 官方 SDK，用于驱动和配置 RealSense 相机

## 关联笔记
- 01KJC0BBHHE7CESYAYN0Z6ZVW5.md: 讨论深度相机选型问题，对比 Gemini 相机的局限性
- 01KJC09C6KBEC83Z6P9EX5Y4F9.md: 调试 USB 双目摄像头的经验记录，涉及 ROS2 相机接入
