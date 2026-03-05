---
note: 01KJC05QF1CDNZ4BMBD9E1R7QK.md
title: 20251015 - 分析ros-humble-isaac-manipulator-bringup的原理 (机械臂复杂流)
indexed_at: 2026-03-05T12:09:43.794162+00:00
---

## 标签
Isaac Sim, ROS2 Humble, 机械臂，pose_to_pose, 运动规划，nvblox

## 摘要
记录了在 Isaac Sim 中运行 ros-humble-isaac-manipulator-bringup 的完整流程，通过 pose_to_pose 工作流实现机械臂在两个目标帧之间的运动规划。解决了 Isaac Sim 5.0 版本兼容性问题（需降级到 4.2）和 QoS 配置问题，最终实现 wrist_3_link 在 target1_frame 和 target2_frame 间交替运动。

## 关键概念
- pose_to_pose: 控制机器人使末端执行器 (wrist_3_link) 到达目标帧 (target1_frame/target2_frame) 位置的工作流
- nvblox: NVIDIA 的 3D 重建库，用于构建环境地图和碰撞检测
- Isaac Sim: NVIDIA 的机器人仿真平台，提供物理仿真和传感器模拟
- 运动规划: cumotion 规划器计算机械臂从起始姿态到目标姿态的无碰撞路径
- ROS2 launch: ROS2 的启动系统，用于配置和启动多个节点的复杂应用

## 关联笔记
- 01KJC05V9SCYYZTVFCHSBJ40S3.md: 后续在 4090 机器上完成 Pick and Place 任务的完整实践，包含 QoS 修改和环境配置
- 01KJC02762W97A18HXBYY70NM7.md: 前期在 Isaac Sim 中进行机械手教程的 Docker 环境配置和安装流程
- 01KJC029N8APNRN6BXH70S5YQD.md: 在两个网络间配置 Isaac Sim 和 ROS2 demo 的教程，包含 SSH 隧道和网络配置
