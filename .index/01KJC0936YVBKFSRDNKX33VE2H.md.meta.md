---
note: 01KJC0936YVBKFSRDNKX33VE2H.md
title: 20251204 - 运行LabUtopia
indexed_at: 2026-03-05T12:17:11.175087+00:00
---

## 标签
LabUtopia, Isaac Sim, ROS2, 机器人仿真, 环境配置, 数据集

## 摘要
记录 LabUtopia 机器人仿真环境的配置和运行流程，包括环境变量设置、依赖安装和包名冲突处理。提供四个关卡（Pick、PourLiquid、HeatLiquid、LiquidMixing）的完整执行命令。

## 关键概念
- LabUtopia: 机器人液体操作任务的数据集和代码框架
- Isaac Sim: NVIDIA 的机器人仿真平台，提供物理引擎和渲染
- ROS2 Humble: 机器人操作系统，用于组件间通信
- RMW CycloneDDS: ROS2 的中间件实现，通过 XML 配置网络行为
- PYTHONPATH: Python 模块搜索路径，用于加载自定义包

## 关联笔记
- 01KJC08TKV281M3RRDK2CJQPCS.md: 包含相同的 ROS2 环境变量配置和 Isaac Sim 路径设置
- 01KJC05V9SCYYZTVFCHSBJ40S3.md: 记录 Isaac Sim 4.2 和 ROS2 开发环境的搭建流程
- 01KJC08NBRWP9P7TG4MVZS84H8.md: 包含类似的容器环境配置和机械臂 ROS2 集成
