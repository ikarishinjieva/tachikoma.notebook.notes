---
note: 01KJBZTYZWC4V68XB7S0KZ986A.md
title: 20250626 - 将Mac OSX和Linux的音频双向转发
indexed_at: 2026-03-05T11:44:53.093152+00:00
---

## 摘要
记录在 Mac OSX 和 Linux 之间建立双向音频转发的完整配置步骤，使用 PulseAudio 的 tunnel 模块实现跨系统音频流传输。通过 SSH 端口转发建立安全通道，配置 module-native-protocol-tcp 和 module-tunnel-sink 实现双向音频路由。

## 关键概念
- PulseAudio: 跨平台音频服务器，支持网络音频流传输
- module-tunnel-sink: PulseAudio 模块，用于将音频输出隧道到远程服务器
- module-loopback: PulseAudio 模块，用于将输入源回环到输出设备
- SSH 端口转发: 通过 SSH 建立加密通道转发 TCP 端口流量

## 关联笔记
- 01KJC029N8APNRN6BXH70S5YQD.md: 同样涉及在 Mac 和 Linux 间建立 SSH 隧道配置网络通道
- 01KJBZTE7KCAE5K5HXHQ091CBR.md: 记录 Mac OSX 重装和 MacPorts 安装，与本文 Mac 端环境配置相关
