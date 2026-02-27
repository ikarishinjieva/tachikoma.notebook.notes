---
note: 01KJC02BRZ0WYFF71HAK5SAJGA.md
title: 20251008 - 列出ROS2样例表, 尝试ROS2样例isaac_ros_apriltag, 解决OOM报错
indexed_at: 2026-02-27T11:48:30.199853+00:00
---

用户要求我分析一篇关于 ROS2 和 Isaac Sim 的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 标题：20251008 - 列出 ROS2 样例表，尝试 ROS2 样例 isaac_ros_apriltag, 解决 OOM 报错
2. 内容主要涉及：
   - Isaac Sim 和 ROS2 环境配置
   - OOM 问题
   - Isaac ROS 学习路径（AprilTag、DNN 检测、SGM 立体视觉、Nvblox、Visual SLAM）
   - 具体尝试 isaac_ros_apriltag 的配置过程

3. 其他笔记列表都是 2021 年的，主题涉及 MySQL、Clickhouse、数据库、性能分析等，与这篇 ROS2/Isaac Sim 的笔记没有明显关联

现在我需要生成：
- 标签（3-6 个）
- 摘要（2-3 句话）
- 关键概念（3-5 个）
- 关联笔记（0-3 篇，看起来没有关联）
## 标签
ROS2, Isaac Sim, AprilTag, 机器人仿真, OOM 问题, 学习路径

## 摘要
记录 Isaac Sim 与 ROS2 环境配置过程，整理从易到难的最小学习路径（AprilTag→DNN 检测→SGM 立体视觉→Nvblox→Visual SLAM）。尝试运行 isaac_ros_apriltag 样例并记录启动配置步骤。

## 关键概念
- AprilTag Detection: 基于视觉标记的检测与位姿估计，无需 DNN 推理
- NITROS: NVIDIA ISAAC ROS 的数据流格式，用于高效节点间通信
- SGM Stereo Disparity: 传统立体匹配算法，无需 DNN 即可生成视差图
- Nvblox: 将相机/深度数据转换为 3D TSDF/ESDF 距离场的建图模块
- QoS: ROS2 服务质量策略，影响话题通信的可靠性与实时性

## 关联笔记
无
