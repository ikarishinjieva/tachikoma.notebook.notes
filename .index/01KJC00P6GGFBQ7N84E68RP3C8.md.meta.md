---
note: 01KJC00P6GGFBQ7N84E68RP3C8.md
title: 20250926 - 搭建Issac环境
indexed_at: 2026-02-27T11:47:12.786242+00:00
---

用户要求我分析一篇关于搭建 Isaac 环境的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 主要内容包括：
   - 配置 NVIDIA Docker
   - 启动 Isaac Sim 容器
   - 尝试远程连接遇到的问题（A100 GPU 不支持 livestreaming，换 4090 仍有错误）
   - 尝试安装 workstation 版本
   - 最终使用阿里云无影云电脑

2. 标签应该是：Isaac Sim, NVIDIA Docker, 容器配置，GPU, 远程连接，云电脑

3. 摘要：记录搭建 Isaac Sim 环境的过程，包括 Docker 容器和 Workstation 两种安装方式的尝试。因 GPU 直播流兼容性问题，最终采用阿里云无影云电脑方案。

4. 关键概念：
   - NVIDIA Container Toolkit: 用于在 Docker 容器中启用 GPU 支持
   - Isaac Sim: NVIDIA 的机器人仿真平台
   - Livestreaming: Isaac Sim 的远程串流功能
   - NVENC: NVIDIA 硬件编码器，A100 GPU 不支持

5. 关联笔记：查看提供的笔记列表，都是关于 MySQL、Clickhouse、数据库、性能分析等内容，与这篇 Isaac Sim 的笔记没有明显关联。所以应该写「无」。

现在按格式输出。
## 标签
Isaac Sim, NVIDIA Docker, GPU 容器，远程连接，环境搭建，云电脑

## 摘要
记录搭建 Isaac Sim 环境的完整过程，包括 NVIDIA Docker 配置和容器启动步骤。因 A100 GPU 不支持直播流功能，尝试 Workstation 版本仍未解决远程连接问题，最终采用阿里云无影云电脑方案。

## 关键概念
- NVIDIA Container Toolkit: 使 Docker 容器能够访问 GPU 资源的工具
- Isaac Sim: NVIDIA 推出的机器人仿真和开发平台
- NVENC: NVIDIA 硬件视频编码器，A100 GPU 不支持此功能
- Livestreaming: Isaac Sim 的远程串流功能，用于客户端连接

## 关联笔记
无
