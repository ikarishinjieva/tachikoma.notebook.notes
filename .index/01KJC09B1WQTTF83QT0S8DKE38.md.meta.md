---
note: 01KJC09B1WQTTF83QT0S8DKE38.md
title: 20251211 - 仿真实验室演示记录
indexed_at: 2026-02-27T11:54:23.777351+00:00
---

用户希望我分析这篇笔记并生成索引元数据。让我先理解笔记内容：

1. 标题：20251211 - 仿真实验室演示记录
2. 内容包含：
   - 多器具陈列（lab-077）
   - 有缓存的实验室场景（034, 040, 085, 023）
   - 环境变量配置（ROS2、Isaac Sim、CUDA 等）
   - 任务命令（运行 HeatLiquid 和 LiquidMixing 仿真）

这是关于机器人仿真/Isaac Sim 实验室的演示记录，涉及 ROS2、Isaac Sim 4.2、CUDA 等技术。

其他笔记列表主要是关于数据库（MySQL、ClickHouse）、性能分析、Java 等相关内容，与这篇仿真实验室笔记关联度较低。

让我生成索引：

标签：仿真、Isaac Sim、ROS2、实验室、演示、环境变量

摘要：记录仿真实验室演示的配置和运行命令。包含多器具陈列和缓存场景的实验室编号，以及 Isaac Sim 4.2 的环境变量和任务执行脚本。

关键概念：
- Isaac Sim: NVIDIA 的机器人仿真平台
- ROS2: 机器人操作系统第二代
- RMW: ROS 中间件实现
- CUDA: NVIDIA 并行计算平台
- ROS_DOMAIN_ID: ROS2 网络隔离标识

关联笔记：无明显关联（其他笔记都是数据库和性能分析相关）
## 标签
仿真、Isaac Sim、ROS2、实验室、演示、环境变量

## 摘要
记录仿真实验室演示的配置信息和运行命令。包含多器具陈列和缓存场景的实验室编号，以及 Isaac Sim 4.2 的环境变量配置和 HeatLiquid、LiquidMixing 两个仿真任务的执行脚本。

## 关键概念
- Isaac Sim: NVIDIA 的机器人仿真平台，版本 4.2
- ROS2: 机器人操作系统第二代，使用 humble 发行版
- RMW: ROS 中间件实现，此处使用 cyclonedds_cpp
- ROS_DOMAIN_ID: ROS2 网络隔离标识，用于区分不同仿真域
- CUDA_VISIBLE_DEVICES: 指定 GPU 设备，此处使用 7 号卡

## 关联笔记
无
