---
note: 01KJC08TKV281M3RRDK2CJQPCS.md
title: 20251125 - 重新梳理机械臂的运行环境初始化命令
indexed_at: 2026-03-05T12:15:34.845505+00:00
---

## 摘要
记录 slider 和 moveit 两台机器的远程登录、容器启动及 ROS2 环境初始化命令。包含完整的代理配置、DDS 设置、CUDA 环境变量及启动脚本。

## 关键概念
- ROS_DOMAIN_ID: ROS2 网络隔离标识，用于区分不同机器人系统
- RMW_IMPLEMENTATION: ROS2 中间件实现，此处使用 Cyclone DDS
- ISAAC_ROS_WS: NVIDIA Isaac ROS 开发工作空间路径
- CYCLONEDDS_URI: Cyclone DDS 配置文件路径，定义网络通信参数

## 关联笔记
- 01KJC09C6KBEC83Z6P9EX5Y4F9.md: 同一项目的 slider/moveit 调试笔记，包含双目摄像头标定和启动脚本配置
- 01KJC09VMCC3G9MT6AZPAWAB54.md: 机械臂/手 AI 示教项目，使用相同的 Isaac ROS 环境和硬件平台
