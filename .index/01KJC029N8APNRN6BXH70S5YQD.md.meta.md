---
note: 01KJC029N8APNRN6BXH70S5YQD.md
title: 20251004 - 在两个网络间，配置Isaac Sim和ROS2 demo, 进行机械手的教程
indexed_at: 2026-02-27T11:48:02.236853+00:00
---

用户要求我分析这篇关于网络配置和 ROS2 的笔记，生成索引元数据。让我仔细阅读笔记内容：

笔记主要内容包括：
1. 实验1 - 在 Mac 和 Linux 间建立 UDP 通道（使用 SSH TUN 隧道）
2. 实验2 - 在 Linux 和 Linux 间建立 UDP 通道
3. 实验3 - 验证 ROS 连接（使用 Docker 和 Fast DDS）

核心主题：网络隧道配置、SSH TUN、UDP 通信、ROS2、DDS

我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇，从提供的列表中找关联）

看关联笔记列表，主要是关于 MySQL、Clickhouse、数据库、性能分析等内容，与这篇网络/ROS2 的笔记没有明显关联。

让我生成输出：## 标签
SSH 隧道，TUN 设备，UDP 通信，ROS2，网络配置，DDS

## 摘要
本笔记记录了在跨网络环境下配置 SSH TUN 隧道的三种实验场景：Mac-Linux 间、Linux-Linux 间建立 UDP 通信通道。同时包含在 Docker 中部署 ROS2 Fast DDS 发现服务器的验证步骤。

## 关键概念
- SSH TUN 隧道: 利用 SSH 的-w 参数创建点对点网络隧道设备
- TUN 设备: 内核提供的三层网络隧道接口，用于路由 IP 数据包
- PermitTunnel: sshd_config 配置项，控制是否允许 SSH 隧道连接
- Fast DDS: ROS2 使用的数据分发服务，用于节点间发现与通信
- 点对点地址: /30 网段的两地址配置，用于隧道两端直接通信

## 关联笔记
无
