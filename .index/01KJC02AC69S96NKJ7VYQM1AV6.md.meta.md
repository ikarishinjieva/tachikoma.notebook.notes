---
note: 01KJC02AC69S96NKJ7VYQM1AV6.md
title: 20251006 - 尝试将Isaac Sim和WebRTC连通
indexed_at: 2026-03-05T12:03:21.968834+00:00
---

## 标签
Isaac Sim, WebRTC, 网络诊断, VPN, 流媒体，跨网络通信

## 摘要
记录了在 WiFi 子网内和通过 VPN 两种场景下测试 Isaac Sim WebRTC 连通性的实验。同子网测试成功，通过 VPN 跨子网连接失败并出现断连错误。使用 tcpdump 和 tshark 进行网络抓包分析，发现跨子网时存在 NAT 宣告 IP 问题。

## 关键概念
- WebRTC: 实时通信协议，用于 Isaac Sim 的流媒体传输
- tcpdump/tshark: 网络抓包和协议分析工具
- carb.livestream-rtc.plugin: Isaac Sim 的直播流 RTC 插件
- NAT: 网络地址转换，跨子网通信的障碍

## 关联笔记
- 01KJC00P6GGFBQ7N84E68RP3C8.md: 搭建 Isaac Sim 环境的基础配置和 livestreaming 文档参考
- 01KJC029N8APNRN6BXH70S5YQD.md: 配置 SSH TUN 隧道连接两个网络的详细教程
- 01KJC02C64360M94KXFDRTPGCM.md: 总结 Isaac Sim 和 ROS2 demo 的探索过程，引用了本笔记的跨子网实验
