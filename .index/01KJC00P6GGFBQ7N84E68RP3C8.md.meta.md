---
note: 01KJC00P6GGFBQ7N84E68RP3C8.md
title: 20250926 - 搭建Issac环境
indexed_at: 2026-03-05T12:00:26.684712+00:00
---

## 摘要
记录在 Linux 服务器上搭建 Isaac Sim 5.0.0 环境的完整过程，包括 NVIDIA Container Toolkit 配置和 Docker 容器启动。尝试容器版和 Workstation 版本均因 GPU 编码和 CUDA 库问题无法远程连接，最终采用阿里云无影云电脑方案。

## 关键概念
- NVIDIA Container Toolkit: 使 Docker 容器能够访问和使用 NVIDIA GPU 的工具
- Isaac Sim Container: NVIDIA 官方提供的容器化仿真环境部署方式
- Livestreaming: Isaac Sim 的远程串流功能，依赖 NVENC 编码器
- NVENC: NVIDIA 硬件视频编码器，A100 GPU 不支持此功能

## 关联笔记
- 01KJC06PG4RM9T7WFDEN5HPZPN.md: 在 Isaac Sim 仿真环境中完成机械臂倒水动作的详细教程
- 01KJC08TKV281M3RRDK2CJQPCS.md: 机械臂运行环境初始化命令，包含 isaac-sim 和 ROS2 配置
