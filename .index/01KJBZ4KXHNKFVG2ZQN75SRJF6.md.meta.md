---
note: 01KJBZ4KXHNKFVG2ZQN75SRJF6.md
title: 20240101 - 构建项目管理chatbot
indexed_at: 2026-03-05T09:25:19.706468+00:00
---

## 标签
chatbot, LangChain, FAISS, 项目管理，向量存储，对话管理

## 摘要
记录使用 LangChain 和 FAISS 向量存储构建项目管理 chatbot 的实现过程，包括聊天记录存储结构和系统架构更新。分析了 GPT-3.5 中对话历史干扰问题，提出使用独立 prompt 重写问题的解决方案。

## 关键概念
- 向量存储记忆: 使用 FAISS 外置存储聊天记录，每个问答对作为独立 Document 存储
- info_or_question 分流器: 区分 COMMON_QUESTION、PROJECT_QUESTION、PROJECT_INFO、COMMON_INFO 四种对话类型
- date_parse: 将模糊日期（如"明天"）转换为具体日期的解析模块
- 问题重写: 将依赖对话历史的问题转换为独立完整的问题，避免历史干扰

## 关联笔记
- 01KJBZ52DM2FY8QMKH4WQ972HT.md: 同系列笔记，分析 LangChain 的 ConversationEntityMemory 记忆机制
- 01KJBZ68S0R9NS2HK7MWN06G5A.md: 涉及 FAISS 向量存储的 embedding 导入和评估
