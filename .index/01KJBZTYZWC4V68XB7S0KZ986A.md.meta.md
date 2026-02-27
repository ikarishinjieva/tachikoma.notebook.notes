---
note: 01KJBZTYZWC4V68XB7S0KZ986A.md
title: 20250626 - 将Mac OSX和Linux的音频双向转发
indexed_at: 2026-02-27T11:41:05.145522+00:00
---

用户希望我分析这篇关于 Mac OSX 和 Linux 音频双向转发的笔记，并生成索引元数据。让我仔细阅读笔记内容，然后按照指定格式输出。

笔记内容分析：
1. 主要讲述如何在 Mac OSX 和 Linux 之间建立双向音频转发
2. 使用 PulseAudio 作为音频服务器
3. 涉及 SSH 隧道、端口转发等技术
4. 包含配置文件修改、服务启动等步骤

标签应该包括：音频转发、PulseAudio、Mac OSX、Linux、SSH 隧道、双向音频

摘要：这篇笔记记录了在 Mac OSX 和 Linux 系统之间建立双向音频转发的完整步骤。通过 PulseAudio 和 SSH 隧道实现两个系统间的音频输入输出互通。

关键概念：
- PulseAudio: 跨平台音频服务器，用于处理音频流转发
- SSH 隧道: 通过加密通道转发音频端口
- module-tunnel-sink: PulseAudio 模块，用于连接远程音频服务器
- module-native-protocol-tcp: 启用 TCP 网络音频协议

关联笔记：浏览其他笔记列表，这些都是关于数据库、MySQL、Clickhouse、系统性能等技术主题，与音频转发没有直接关联，所以应该写「无」。## 标签
音频转发，PulseAudio，Mac OSX，Linux，SSH 隧道，双向音频

## 摘要
这篇笔记记录了在 Mac OSX 和 Linux 系统之间建立双向音频转发的完整配置步骤。通过 PulseAudio 音频服务器配合 SSH 端口转发，实现两个系统间的音频输入输出互通。

## 关键概念
- PulseAudio: 跨平台音频服务器，用于处理和转发音频流
- SSH 隧道: 通过加密通道转发音频端口实现跨系统连接
- module-tunnel-sink: PulseAudio 模块，用于连接远程音频服务器创建虚拟输出设备
- module-native-protocol-tcp: 启用 TCP 网络音频协议允许远程客户端连接

## 关联笔记
无
