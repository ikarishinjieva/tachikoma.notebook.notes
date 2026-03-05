---
note: 01KJC05V9SCYYZTVFCHSBJ40S3.md
title: 20251017 - 为机械臂开发环境, 整备4090 48G机器, 并完成Pick and Place任务
indexed_at: 2026-03-05T12:09:59.441854+00:00
---

## 摘要
记录在 4090 48G 服务器上配置 Isaac Sim 4.2 和 ROS2 Humble 开发环境的全过程，包括代理设置、容器配置和模型下载。成功执行机械臂 Pick and Place 任务，解决了 QoS 匹配、ESDF 数据和 TF 坐标变换等问题。

## 关键概念
- NICE DCV: 远程桌面协议，用于在服务器上运行图形化 Isaac Sim 界面
- QoS (Quality of Service): ROS2 通信质量策略，影响消息传输的可靠性
- cuMotion: NVIDIA 的运动规划器，用于机械臂路径规划和避障
- ROS2 Bridge: Isaac Sim 与 ROS2 之间的通信桥接层

## 关联笔记
- 01KJC02E4GN93SJMVEZQJ82A57.md: 环境安装和运行指令的原始参考文档
- 01KJC05QF1CDNZ4BMBD9E1R7QK.md: 机械臂复杂流的原理分析，本次任务跟随的案例
