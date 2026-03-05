---
note: 01KJBZA9GKXAFRZ60P2VWBRN2C.md
title: 20240621 - ChatDBA: 意图识别错误, 尝试用问题重写解决
indexed_at: 2026-03-05T10:14:20.168407+00:00
---

## 摘要
记录 ChatDBA 将用户主从数据异常问题误判为故障诊断而非原理解释的问题，尝试通过问题重写方案解决意图识别错误。

## 关键概念
- 意图识别: 判断用户对话意图的分类系统（故障诊断/名词解释/指令生成/工具解释）
- 问题重写: 将用户问题改写成更清晰或更易于 LLM 理解的形式
- 主题管理: 管理对话主题列表及主题切换的工具系统

## 关联笔记
- 01KJBZADENS4HHC18MT0EAZ182.md: 提到 topic_manager 使用 LLM 答案进行意图识别而非话题管理
- 01KJBZR8XAJASCJPA6NVR9TQBQ.md: 定义问题重写 (Query Rewriting) 作为优化 LLM 理解的方案
- 01KJBZAHH8971RB2AQ5TZRFTPA.md: 提出在问题重写时对问题进行拆解的替代方案
