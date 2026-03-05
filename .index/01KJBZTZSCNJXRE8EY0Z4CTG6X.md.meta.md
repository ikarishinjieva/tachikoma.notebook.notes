---
note: 01KJBZTZSCNJXRE8EY0Z4CTG6X.md
title: 20250627 - cursor使用经验
indexed_at: 2026-03-05T11:45:13.451932+00:00
---

## 摘要
记录使用 Cursor 进行 AI 辅助开发的经验总结，包括需求管理、代码分析、功能迭代等最佳实践。强调分阶段开发（分析→设计→实现）和维护 code-analyze 文档（界面-API-功能流程 - 代码函数对应关系）的重要性。

## 关键概念
- code-analyze.txt: 维护界面-API-主要功能流程 - 主要代码函数对应关系的文档
- 状态无关界面: 用唯一数据结构维持界面所有数据，使 AI 易识别和维护
- 分阶段开发: 功能设计→简化→代码规划→实现，各阶段职责分离不越界
- 模拟模式: 由 AI 函数进行模拟，而非让大模型自定模拟端

## 关联笔记
- 01KJC0BM188531P5364MW0W802.md: 配置 Cursor MCP 浏览器功能的实践记录
- 01KJC0GPN5FTPTV5X28W3WCVYW.md: 使用 Cursor superpowers 和 MCP 进行项目开发的经验
