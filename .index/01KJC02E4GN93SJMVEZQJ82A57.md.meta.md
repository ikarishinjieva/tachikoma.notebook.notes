---
note: 01KJC02E4GN93SJMVEZQJ82A57.md
title: 20251010 - 整理Isaac Sim和ROS2 demo的环境安装和运行指令
indexed_at: 2026-02-27T11:48:52.377812+00:00
---

用户希望我分析这篇关于 Isaac Sim 和 ROS2 环境安装的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 主题：使用公司服务器安装 Isaac Sim 和 ROS2 demo，本地笔记本远程连接
2. 主要内容：配置 NICE DCV 远程桌面连接的详细步骤
3. 涉及的技术：NICE DCV、GPU 共享、OpenGL、NVIDIA 驱动等

从其他笔记列表来看，大部分是关于 MySQL、Clickhouse、数据库相关的笔记，与这篇 Isaac Sim/ROS2/DCV 的内容没有直接关联。

让我生成索引：

标签：Isaac Sim, ROS2, NICE DCV, 远程桌面, GPU 加速, Ubuntu

摘要：记录在公司服务器上安装 Isaac Sim 和 ROS2 demo 的环境配置过程。重点描述了 NICE DCV 远程桌面服务的安装配置，包括 GPU 共享和硬件加速的验证方法。

关键概念：
- NICE DCV: 远程桌面协议，支持 GPU 共享的虚拟会话
- DCV GL: NICE DCV 的 GPU 加速组件，用于硬件渲染
- glxinfo: 用于检查 OpenGL 渲染器和 GPU 加速状态的工具
- 显示管理器 (gdm3): Ubuntu 桌面环境的显示管理服务

关联笔记：无（其他笔记都是数据库相关的，与这篇内容无关）
## 标签
Isaac Sim, ROS2, NICE DCV, 远程桌面, GPU 加速, Ubuntu

## 摘要
记录在公司服务器上安装 Isaac Sim 和 ROS2 demo 的环境配置过程。重点描述了 NICE DCV 远程桌面服务的安装配置，包括 GPU 共享和硬件加速的验证与问题排查方法。

## 关键概念
- NICE DCV: 支持 GPU 共享的远程桌面协议，用于虚拟会话访问
- DCV GL: NICE DCV 的 GPU 加速组件，确保硬件渲染而非 Mesa 软解
- glxinfo: 用于检查 OpenGL 渲染器和 GPU 加速状态的工具
- 显示管理器 (gdm3): Ubuntu 桌面环境的显示管理服务，DCV 依赖其初始化

## 关联笔记
无
