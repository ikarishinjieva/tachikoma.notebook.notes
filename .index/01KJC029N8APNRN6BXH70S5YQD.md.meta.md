---
note: 01KJC029N8APNRN6BXH70S5YQD.md
title: 20251004 - 在两个网络间，配置Isaac Sim和ROS2 demo, 进行机械手的教程
indexed_at: 2026-03-05T12:02:32.077623+00:00
---

## 标签
SSH 隧道，UDP 通道，ROS2，Isaac Sim，网络配置，跨平台通信

## 摘要
记录在 macOS-Linux 和 Linux-Linux 间建立 SSH TUN 隧道进行 UDP 通信的完整实验步骤。包含隧道接口 IP 配置、连通性验证方法，以及 ROS2 发现服务器的 Docker 部署与连接测试流程。

## 关键概念
- SSH TUN 隧道: 通过 ssh -w 参数创建点对点网络隧道设备，实现跨网络通信
- PermitTunnel: sshd_config 配置项，控制服务器是否允许创建 TUN 设备
- 点对点 IP 配置: 为隧道两端分配同一网段的/30 地址，建立直接通信链路
- Fast DDS 发现服务器: ROS2 用于服务发现的组件，支持单播模式跨网络通信

## 关联笔记
- 01KJC02E4GN93SJMVEZQJ82A57.md: Isaac Sim 和 ROS2 demo 的完整环境安装与运行指令
- 01KJC02C64360M94KXFDRTPGCM.md: Isaac Sim 和 ROS2 demo 的探索总结，引用本笔记的网络连接方案
