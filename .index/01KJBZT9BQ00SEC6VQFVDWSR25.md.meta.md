---
note: 01KJBZT9BQ00SEC6VQFVDWSR25.md
title: 20250601 - 直播音频生成脚本
indexed_at: 2026-02-27T11:39:43.710080+00:00
---

用户让我分析一篇关于"直播音频生成脚本"的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一个 Python 脚本（1.py）
2. 使用 Flask 和 WebSocket
3. 调用讯飞开放平台的 TTS（Text-to-Speech）API
4. 功能是生成音频文件
5. 包含 API 配置、WebSocket 认证、TTS 转换类等

现在我需要生成：
1. 标签（3-6 个）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

看其他笔记列表，大部分是关于 MySQL、Clickhouse、数据库相关的，这篇笔记是关于 TTS/音频生成的，看起来没有直接关联的笔记。

让我生成元数据：

标签：Python, TTS, 讯飞 API, Flask, WebSocket, 音频生成

摘要：这是一个使用讯飞开放平台 TTS API 的 Python 脚本，通过 WebSocket 连接将文本转换为音频。脚本使用 Flask 提供 Web 服务，并存储生成的脚本和音频文件。

关键概念：
- TTS (Text-to-Speech): 文本转语音技术
- WebSocket 认证: 使用 HMAC-SHA256 生成认证 URL
- Flask: Python Web 框架，用于提供 HTTP 服务


- 讯飞开放平台: 提供语音合成等 AI 能力的平台
- HMAC-SHA256: 用于生成 API 签名的加密算法

关联笔记：无（其他笔记都是数据库相关的，与这篇 TTS/音频生成没有直接关联）
## 标签
Python, TTS, 讯飞 API, Flask, WebSocket, 音频生成

## 摘要
这是一个使用讯飞开放平台 TTS API 的 Python 脚本，通过 WebSocket 连接将文本转换为音频。脚本使用 Flask 提供 Web 服务，并存储生成的脚本和音频文件到本地目录。

## 关键概念
- TTS (Text-to-Speech): 文本转语音技术，将文字转换为音频
- WebSocket 认证: 使用 HMAC-SHA256 算法生成带签名的认证 URL
- Flask: Python 轻量级 Web 框架，用于提供 HTTP 服务接口
- 讯飞开放平台: 提供语音合成等 AI 能力的云平台

## 关联笔记
无
