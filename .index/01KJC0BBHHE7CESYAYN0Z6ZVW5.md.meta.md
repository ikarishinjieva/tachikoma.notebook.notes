---
note: 01KJC0BBHHE7CESYAYN0Z6ZVW5.md
title: 20260122 - 关于深度相机的选择
indexed_at: 2026-03-05T12:21:51.859060+00:00
---

## 标签
深度相机, gemini 相机, ROS2, 相机标定, D2C, 传感器选型

## 摘要
记录 gemini 深度相机使用中遇到的技术问题，包括 camera_info 缺少时间戳、光学坐标系不符合 ROS2 规范、硬件 D2C 分辨率限制等。提出相机选型时需考虑 IMU/里程计集成能力。

## 关键概念
- camera_info: ROS2 相机内参消息，包含标定参数和时间戳
- 光学坐标系: ROS2 规范定义的相机坐标系（X-右，Y-下，Z-前）
- D2C (Depth to Color): 深度图与彩色图对齐的硬件处理能力
- 硬件 D2C: 相机内部实现的深度 - 彩色对齐，分辨率受限

## 关联笔记
- 01KJC09HGP5EADCGYKZC5QCW6M.md: 记录 gemini max 相机测试，同样提到 camera_info 时间戳为 0 的问题及修复方案
- 01KJC09C6KBEC83Z6P9EX5Y4F9.md: 详细说明相机光学坐标系验证方法和手眼标定流程
- 01KJC09P0B1PQVA3T49QEWQ6RF.md: 记录 D2C 对齐偏差问题和坐标系适配经验
