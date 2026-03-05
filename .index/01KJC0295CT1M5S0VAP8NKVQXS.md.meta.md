---
note: 01KJC0295CT1M5S0VAP8NKVQXS.md
title: 20251002 - 用移动硬盘安装第二个操作系统ubuntu
indexed_at: 2026-03-05T12:01:42.853491+00:00
---

## 标签
Ubuntu, 外置硬盘, UEFI 引导，双系统，GPT 分区，系统安装

## 摘要
完整记录将 Ubuntu 操作系统安装到外置 SSD/移动硬盘的全流程，包括分区配置、引导器设置和风险控制。实现插盘启动 Ubuntu、拔盘自动返回 Windows 的双系统方案，不影响内置硬盘。

## 关键概念
- UEFI 引导：现代计算机固件启动标准，支持从外置设备独立引导操作系统
- GPT 分区：全局唯一标识分区表，UEFI 启动必需的磁盘分区格式
- EFI 系统分区 (ESP)：存放引导装载器的 FAT32 分区，本方案中位于外置盘
- 手动分区：安装时选择"其他选项"精确控制分区位置，避免写入内置硬盘

## 关联笔记
- 01KJC08NBRWP9P7TG4MVZS84H8.md: 记录为机械臂配置笔记本并安装 Ubuntu 22.04 的实操过程
- 01KJC02AC69S96NKJ7VYQM1AV6.md: 涉及 Ubuntu server 环境下 NICE DCV 远程桌面的安装配置
- 01KJC0BFT50E4S8YYFV2246EWD.md: 记录 Ubuntu 22.04 系统上 NICE DCV 软件的安装与调试
