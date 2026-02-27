---
note: 01KJC02AYWC5THW8GV1H8Y7AE7.md
title: 20251007 - 诊断Isaac Sim和ros2 launch运行时发生OOM
indexed_at: 2026-02-27T11:48:17.852354+00:00
---

用户要求我分析这篇关于 Isaac Sim 和 ROS2 launch 运行时发生 OOM（内存溢出）问题的笔记，并生成索引元数据。

让我分析笔记内容：
1. 主要问题：Isaac Sim 和 ROS2 launch 同时运行时发生 OOM（内存溢出）
2. 尝试的解决方案：禁用组件、修改环境变量、清理共享内存、调整场景等
3. 发现：使用 GT 视觉代替真实视觉不会 OOM，但启用动态组件后会 OOM
4. 结论：OOM 可能与某个不定期触发的机制有关

标签应该是：Isaac Sim, ROS2, OOM, 内存问题, 调试

摘要：记录 Isaac Sim 与 ROS2 launch 同时运行时发生 OOM 的排查过程。尝试多种方案均无效，发现与动态组件和视觉系统有关。

关键概念：
- OOM: 内存溢出问题
- Isaac Sim: NVIDIA 的机器人仿真平台
- ROS2: 机器人操作系统
- 动态组件: 运行时加载的功能模块

关联笔记：从列表中看，01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 可能与内存问题相关

让我生成输出。## 标签
Isaac Sim, ROS2, OOM, 内存问题, 调试, 动态组件

## 摘要
记录 Isaac Sim 与 ROS2 launch 同时运行时发生 OOM（内存溢出）的完整排查过程。尝试禁用组件、调整环境变量、清理共享内存、修改场景等多种方案均无效，最终发现 OOM 与动态组件（front hawk manipulator）和视觉系统的不定期触发机制有关。

## 关键概念
- OOM: 内存溢出（Out of Memory），服务器 200G+ 内存被完全占用
- Isaac Sim: NVIDIA 的机器人仿真平台，与 ROS2 集成运行
- 动态组件: 运行时加载的功能模块，如机械臂、视觉渲染等
- RMW_IMPLEMENTATION: ROS2 的中间件实现，此处使用 rmw_fastrtps_cpp
- ROS_DOMAIN_ID: ROS2 网络隔离标识，用于区分不同通信域

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（同为内存问题排查主题）
