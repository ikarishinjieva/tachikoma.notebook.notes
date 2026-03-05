---
note: 01KJC08NBRWP9P7TG4MVZS84H8.md
title: 20251107 - 为机械臂配置笔记本
indexed_at: 2026-03-05T12:14:05.636867+00:00
---

## 标签
机械臂，笔记本配置，Ubuntu 22.04，ROS2，NICE DCV，Docker

## 摘要
记录为小象机械臂（myCobot 280）配置专用笔记本的完整过程，包括 Ubuntu 22.04 安装、Docker 配置、ROS2 环境搭建、串口驱动调试及 NICE DCV 远程桌面部署。解决了无显卡笔记本运行 DCV 会话和 Docker 容器配置问题。

## 关键概念
- myCobot 280: 小象机器人六轴机械臂，通过串口 `/dev/ttyACM0` 通信，波特率 115200
- NICE DCV: 远程桌面协议，用于无显卡笔记本创建虚拟会话需先禁用图形界面 (gdm3)
- ROS2 Humble: 机器人操作系统，配合 moveit 进行机械臂运动规划
- pymycobot: 机械臂 Python SDK，用于直接控制机械臂动作和 LED

## 关联笔记
- 01KJC02E4GN93SJMVEZQJ82A57.md: 被引用的 Isaac Sim 和 ROS2 demo 环境安装指令源文档
- 01KJC08TKV281M3RRDK2CJQPCS.md: 后续整理的机械臂运行环境初始化命令，基于本笔记经验
