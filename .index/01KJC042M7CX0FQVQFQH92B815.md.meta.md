---
note: 01KJC042M7CX0FQVQFQH92B815.md
title: 20251012 - 分析isaac_ros_visual_slam的原理 (机器人视觉感知)
indexed_at: 2026-03-05T12:07:50.732847+00:00
---

## 摘要
记录 isaac_ros_visual_slam (cuVSLAM) 的安装配置与 Isaac Sim 联合实验过程，对比其与 Nvblox 在位姿估计上的区别（Nvblox 使用 GT 位姿，cuVSLAM 需自行估算）。排查 QoS 配置问题导致点云显示失败，解释稀疏点云是 SLAM 用于全局定位的特征点地图而非稠密避障地图。

## 关键概念
- cuVSLAM: NVIDIA 基于 CUDA 加速的稀疏视觉 SLAM 系统，通过提取特征点实现机器人定位
- 稀疏 VSLAM: 只提取几百到几千个稳定特征点构建"骨架地图"，用于定位而非建图
- 位姿估计: 机器人通过匹配当前视野特征点与地图地标来计算自身位置和姿态
- QoS (Quality of Service): ROS2 通信质量策略，配置不当会导致话题订阅失败

## 关联笔记
- 01KJC03VRF6SMQ2J69DXQ5SBTA.md: 分析 isaac_ros_nvblox 的原理，本笔记开头对比了两者在位姿来源上的区别
- 01KJC02BRZ0WYFF71HAK5SAJGA.md: 提到 cuVSLAM 学习路径，包括跑通轨迹、回环和地图保存/加载
