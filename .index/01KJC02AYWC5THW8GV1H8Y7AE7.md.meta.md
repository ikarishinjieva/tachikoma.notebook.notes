---
note: 01KJC02AYWC5THW8GV1H8Y7AE7.md
title: 20251007 - 诊断Isaac Sim和ros2 launch运行时发生OOM
indexed_at: 2026-03-05T12:04:06.873213+00:00
---

## 标签
Isaac Sim, ROS2, OOM, 内存泄漏，故障排查，机械臂仿真

## 摘要
记录 Isaac Sim 与 ROS2 launch 同时运行时发生 OOM 问题的完整排查过程。尝试了禁用多个动态组件、清理共享内存、调整场景等多种方法均未解决，最终发现启用 front hawk manipulator 组件会触发 OOM，且问题与某个不定期触发的机制有关。

## 关键概念
- OOM (Out of Memory): 系统内存耗尽导致进程被终止的故障
- ROS_DOMAIN_ID: ROS2 用于隔离通信的环境变量
- rmw_fastrtps_cpp: ROS2 的 Fast RTPS 中间件实现
- NICE DCV: 远程桌面显示协议，用于无显示器服务器的图形输出

## 关联笔记
- 01KJC02AC69S96NKJ7VYQM1AV6.md: 本笔记引用的启动步骤来源，记录了 Isaac Sim 和 WebRTC 连通尝试及 NICE DCV 配置
- 01KJC02BRZ0WYFF71HAK5SAJGA.md: 同系列 OOM 问题排查笔记，记录了列出 ROS2 样例表并解决 OOM 报错的过程
- 01KJC05V9SCYYZTVFCHSBJ40S3.md: 后续环境整备笔记，在 4090 48G 机器上成功完成 Pick and Place 任务
