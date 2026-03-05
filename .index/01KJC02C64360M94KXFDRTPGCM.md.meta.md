---
note: 01KJC02C64360M94KXFDRTPGCM.md
title: 20251008 - 关于Isaac Sim 和 ROS2 demo的文章总结
indexed_at: 2026-03-05T12:04:48.142476+00:00
---

## 摘要
记录了在 Isaac Sim 与 ROS2 demo 集成过程中遇到的各类环境配置问题和 OOM 错误的排查历程。从 headless 模式尝试、跨网络连接、显卡兼容性问题，到最终通过简化测试场景和切换 DDS 实现突破。

## 关键概念
- fastdds: ROS2 使用的服务发现协议，v2 与 v3 版本配置差异导致跨网络连接问题
- headless 模式: Isaac Sim 无显示器运行模式，需通过 Client 远程连接显示
- OOM (Out of Memory): 显存或内存不足错误，由 fastdds 网络回环、Isaac Sim 缓存等多因素触发
- SSH TUN 通道: 用于跨子网连接 Isaac Sim 和 ROS2 demo 的网络隧道方案
- cyclonedds: 替代 fastdds 的 DDS 实现，可避免某些 OOM 问题

## 关联笔记
- 01KJC02E4GN93SJMVEZQJ82A57.md: 最终整理的 Isaac Sim 和 ROS2 demo 完整安装与运行指令
- 01KJC02BRZ0WYFF71HAK5SAJGA.md: 通过简化样例测试定位 OOM 根源并解决问题
- 01KJC02AYWC5THW8GV1H8Y7AE7.md: 诊断 Isaac Sim 和 ROS2 launch 运行时 OOM 问题的过程记录
