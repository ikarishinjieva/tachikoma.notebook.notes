---
note: 01KJBZEMNJW3ZAV821WQ5N70HB.md
title: 20240822 - 试用 lavague
indexed_at: 2026-03-05T10:37:06.087563+00:00
---

## 摘要
记录在 Ubuntu 服务器上安装配置 LaVague 网页自动化框架的过程，包括 conda 环境创建、依赖安装和代理配置。重点记录了 Chrome/Selenium 启动失败的问题排查，最终通过安装 libgbm1 和 libnss3 依赖解决。

## 关键概念
- LaVague: 基于 LLM 的网页自动化框架，通过自然语言指令控制浏览器操作
- SeleniumDriver: LaVague 的 Selenium 浏览器驱动，用于执行网页交互操作
- WorldModel: LaVague 核心组件，分析屏幕截图并决策下一步操作
- ActionEngine: 将 WorldModel 的决策转换为具体的 Selenium API 调用
- ChromeDriver: Selenium 的 Chrome 浏览器自动化驱动，需要系统依赖支持

## 关联笔记
- 01KJBZEKMQ04Q6EWJA0CPDTVVS.md: 次日对 LaVague 源码结构的深入解析笔记
- 01KJBZEQDEV2ES96F3T90D1B65.md: 讨论 LaVague 截屏方案及改进思路
