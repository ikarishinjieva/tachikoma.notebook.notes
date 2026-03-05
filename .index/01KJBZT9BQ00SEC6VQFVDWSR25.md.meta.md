---
note: 01KJBZT9BQ00SEC6VQFVDWSR25.md
title: 20250601 - 直播音频生成脚本
indexed_at: 2026-03-05T11:42:07.494683+00:00
---

## 标签
Python, Flask, TTS, 讯飞 API, 音频生成, WebSocket

## 摘要
基于讯飞 TTS API 的直播音频生成脚本，使用 Flask 提供 Web 界面和 API 服务。包含 WebSocket 认证、语音合成、音频文件存储和导出功能。

## 关键概念
- TTS (Text-to-Speech): 将文本转换为语音的技术，使用讯飞开放平台 API
- WebSocket 认证: 通过 HMAC-SHA256 签名生成认证的 WebSocket 连接 URL
- Flask Web 服务: 提供音频生成、获取和导出的 HTTP API 接口
- 语音角色: 支持不同音色（如女生、男生）的语音合成选项

## 关联笔记
- 01KJBZT9QYSP83S673RT3276T1.md: 同一天创建的笔记，涉及直播间图片编辑，与直播场景相关
- 01KJBZTCC5H1T0RB7EF3PBCVB8.md: 涉及生成音频计算台词时长，与音频生成用途相关
- 01KJBZTYZWC4V68XB7S0KZ986A.md: 关于 Linux 和 Mac 音频转发，与音频处理相关
