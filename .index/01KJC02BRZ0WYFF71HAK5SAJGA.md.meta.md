---
note: 01KJC02BRZ0WYFF71HAK5SAJGA.md
title: 20251008 - 列出ROS2样例表, 尝试ROS2样例isaac_ros_apriltag, 解决OOM报错
indexed_at: 2026-03-05T12:04:22.086865+00:00
---

## 摘要
记录 Isaac Sim 与 ROS2 集成学习路径（从 AprilTag 检测到 Visual SLAM 的 5 阶段方案），以及尝试运行 isaac_ros_apriltag 样例的具体配置过程和 OOM 问题背景。包含 DCV 远程桌面启动 Isaac Sim 和 ROS2 dev 容器的完整命令。

## 关键概念
- Isaac ROS: NVIDIA 为 ROS2 提供的加速感知包集合，支持 TensorRT 推理和 NITROS 数据格式
- AprilTag Detection: 基于 fiducial marker 的视觉定位技术，用于获取相机与标记物的相对位姿
- NITROS: NVIDIA ROS 2 优化数据格式，通过零拷贝提升相机/深度等数据流性能
- SGM Stereo: 半全局匹配双目立体视觉算法，无需 DNN 模型即可生成视差/深度图
- Nvblox: 基于 TSDF/ESDF 的 3D 建图库，用于导航避障的距离场表示

## 关联笔记
- 01KJC02AC69S96NKJ7VYQM1AV6.md: 本文引用的 DCV 配置和 Isaac Sim 启动流程的原始文档
- 01KJC05QF1CDNZ4BMBD9E1R7QK.md: 同系列笔记，记录 isaac_manipulator pick_and_place 样例的 OOM 问题调试过程
- 01KJC05V9SCYYZTVFCHSBJ40S3.md: Isaac Sim 和 ROS2 demo 的环境安装指令整理，包含 QoS 配置等解决方案
