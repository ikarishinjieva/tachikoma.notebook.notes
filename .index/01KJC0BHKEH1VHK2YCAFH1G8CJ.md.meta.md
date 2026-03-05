---
note: 01KJC0BHKEH1VHK2YCAFH1G8CJ.md
title: 20260203 - 搭建openclaw服务器
indexed_at: 2026-03-05T12:22:27.245060+00:00
---

## 摘要
记录在 Ubuntu 服务器上通过一键脚本安装 OpenClaw 的全过程。包含代理配置、apt 依赖包冲突问题的解决方法及 SSH 端口转发访问 Web UI 的配置。

## 关键概念
- OpenClaw: AI 代理框架，提供 Web Dashboard 和 API 控制界面
- SSH 端口转发: 通过 `ssh -N -L` 将远程服务的本地端口映射到本地浏览器
- apt 依赖修复: 使用 `apt --fix-broken install` 解决包版本不匹配问题
- 环境变量代理: 通过 `http_proxy/https_proxy` 配置网络代理访问外部资源
- Dashboard Token: OpenClaw Web UI 的认证令牌，用于安全访问

## 关联笔记
- 01KJC0BM188531P5364MW0W802.md: 同一天的服务器配置笔记，使用相同的代理设置 (10.186.16.135:7890)
- 01KJC0GNEWJE16AKQPJJFFD1FX.md: 类似的 AI 工具一键安装流程，安装模式相同
- 01KJC05V9SCYYZTVFCHSBJ40S3.md: UCloud 服务器环境配置，涉及相同的代理和网络配置
