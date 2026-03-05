---
note: 01KJC02E4GN93SJMVEZQJ82A57.md
title: 20251010 - 整理Isaac Sim和ROS2 demo的环境安装和运行指令
indexed_at: 2026-03-05T12:05:04.392150+00:00
---

## 标签
Isaac Sim, ROS2, NICE DCV, 远程桌面, 环境配置, GPU 加速

## 摘要
记录在公司服务器上安装 Isaac Sim 和 ROS2 demo 的完整流程，重点配置 NICE DCV 虚拟会话实现远程桌面连接。包含 DCV GL GPU 加速验证和问题排查（Mesa 与 NVIDIA 驱动冲突）。

## 关键概念
- NICE DCV: Amazon 远程桌面协议，支持 GPU 共享的虚拟会话
- DCV GL: NICE DCV 的 GPU 加速组件，用于 OpenGL 硬件渲染
- 虚拟会话: DCV 的桌面会话模式，支持多用户共享 GPU 资源
- glxinfo: OpenGL 信息查询工具，用于验证 GPU 加速状态

## 关联笔记
- 01KJC05V9SCYYZTVFCHSBJ40S3.md: 引用本笔记作为机械臂开发环境配置的参考
- 01KJC08NBRWP9P7TG4MVZS84H8.md: 按照本笔记配置 ROS2 DEMO 环境
- 01KJC0BFT50E4S8YYFV2246EWD.md: 后续解决 NICE DCV license 问题的记录
