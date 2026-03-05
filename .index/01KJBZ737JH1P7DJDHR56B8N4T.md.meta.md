---
note: 01KJBZ737JH1P7DJDHR56B8N4T.md
title: 20240301 - 学习 LLM Dialogue Management
indexed_at: 2026-03-05T09:39:16.580702+00:00
---

## 摘要
DiagGPT 论文提出了一种基于 GPT-4 的多代理 AI 系统，专为任务导向对话（TOD）场景设计，具备自动主题管理能力。系统核心模块包括 Chat Agent、Topic Manager、Topic Enricher 和 Context Manager，通过四阶段工作流程实现对话主题的自动追踪和管理。笔记记录了系统框架、工作流程及 Chat Agent 提示词设计，展示了如何通过主题栈管理和主动提问引导用户完成复杂诊断任务。

## 关键概念
- Task-Oriented Dialogue (TOD): AI 聊天代理需主动提问并引导用户完成特定任务的对话模式
- Topic Manager: 管理对话主题的核心模块，分析用户查询并预测主题发展方向
- Topic Stack: 维护整个对话的主题栈，追踪对话状态和主题进展
- Chat Agent: 与用户交互的对话代理，根据提示词和当前主题生成响应

## 关联笔记
- 01KJBZ9CGND48QFTK6R3W2QQ97.md: UniMS-RAG 论文笔记，同样探讨多知识来源的个性化对话系统设计
- 01KJBZ5QR6TEBCYB2BPT5AVM4C.md: SURGE 论文笔记，研究知识图谱增强的知识 grounded 对话生成
- 01KJBZ7KPW7MR951Y82WGFBVF7.md: ChatDBA 对话评分系统，涉及任务导向对话的质量评估方法
