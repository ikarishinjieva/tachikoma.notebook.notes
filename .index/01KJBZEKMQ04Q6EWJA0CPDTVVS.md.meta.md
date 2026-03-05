---
note: 01KJBZEKMQ04Q6EWJA0CPDTVVS.md
title: 20240823 - 解析 LaVague
indexed_at: 2026-03-05T10:36:37.830907+00:00
---

## 摘要
这篇笔记分析了 LaVague 框架的核心架构，包括 WebAgent、WorldModel、ActionEngine 及其子模块 NavigationEngine 和 PythonEngine 的工作原理。重点说明了 NavigationEngine 如何通过 HTML 元素提取、RAG 召回和 LLM 决策来实现网页操作。

## 关键概念
- WebAgent: LaVague 的核心代理，由 WorldModel 决策者和 ActionEngine 执行者组成
- NavigationEngine: 通过提取可操作元素、RAG 召回和 LLM 决策来执行网页导航
- PythonEngine: 使用 LLM 分析 HTML 内容，支持多模态 fallback 机制处理置信度不足的情况
- WorldModel: 扮演决策者角色，分析截图并生成指令给其他专用 AI 执行
- InteractiveXPathRetriever: 识别 HTML 中可操作元素并通过注入脚本添加 xpath 属性

## 关联笔记
- 01KJBZEMNJW3ZAV821WQ5N70HB.md: 记录 LaVague 的试用过程和 WebAgent 代码示例
- 01KJBZEQDEV2ES96F3T90D1B65.md: 讨论类似 LaVague 的截全屏和元素定位问题
