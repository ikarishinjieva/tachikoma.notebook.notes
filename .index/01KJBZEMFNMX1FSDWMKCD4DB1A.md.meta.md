---
note: 01KJBZEMFNMX1FSDWMKCD4DB1A.md
title: 20240824 - 测试使用 playwright
indexed_at: 2026-03-05T10:36:54.002702+00:00
---

## 摘要
记录在远程服务器 (10.186.58.23) 上配置 Playwright 浏览器自动化环境的完整步骤。包含 conda 环境创建、依赖安装、X11 转发配置及可运行的测试代码示例。

## 关键概念
- Playwright: 微软开源的浏览器自动化库，支持 Chromium/Firefox/WebKit
- sync_playwright: Playwright 的同步 API 入口，用于创建浏览器会话
- X11 转发: 通过 SSH -XY 参数实现远程图形界面显示到本地

## 关联笔记
- 01KJBZEQDEV2ES96F3T90D1B65.md: 后续开发 ai-tester 项目，明确标注此前置笔记
- 01KJC0GPN5FTPTV5X28W3WCVYW.md: 后续使用 Playwright MCP 进行浏览器测试的进阶实践
