---
note: 01KJC03VRF6SMQ2J69DXQ5SBTA.md
title: 20251011 - 分析isaac_ros_nvblox的原理 (3D地图建立, 以及机器人路径规划)
indexed_at: 2026-03-05T12:07:24.154497+00:00
---

## 标签
isaac_ros_nvblox, 3D 重建，路径规划，Isaac Sim, ROS2, 场景重建

## 摘要
记录 isaac_ros_nvblox 包的安装流程和与 Isaac Sim 配合使用的方法，涵盖 3D 地图建立、多摄像头配置、雷达数据融合及人类识别功能。通过 RViz 可视化 3D 重建结果并使用 2D Goal Pose 实现自动路径规划。

## 关键概念
- nvblox: NVIDIA 的 3D 体素地图重建库，支持 TSDF/ESDF 距离场生成
- TSDF: 截断符号距离场，用于表示物体表面的 3D 几何信息
- ESDF: 欧几里得符号距离场，用于路径规划中的障碍物查询
- ROS2 Navigation: 基于 nvblox 地图进行代价地图融合和路径规划

## 关联笔记
- 01KJC05QF1CDNZ4BMBD9E1R7QK.md: 同样使用 nvblox 进行机械臂场景的 3D 重建和路径规划
- 01KJC02762W97A18HXBYY70NM7.md: 涉及 isaac_ros_nvblox 的安装步骤和 ROS2 容器配置
- 01KJC02AYWC5THW8GV1H8Y7AE7.md: Isaac Sim 与 ROS2 联合运行的环境配置和 OOM 问题诊断
