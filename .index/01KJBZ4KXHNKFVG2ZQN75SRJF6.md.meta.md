---
note: 01KJBZ4KXHNKFVG2ZQN75SRJF6.md
title: 20240101 - 构建项目管理chatbot
indexed_at: 2026-02-27T10:41:43.285044+00:00
---

用户要求我分析这篇关于构建项目管理 chatbot 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：构建项目管理 chatbot
2. 主要内容：
   - 使用向量存储（FAISS）记录聊天记录
   - LangChain 的 vectorstore retriever memory
   - 20240131 更新：新的架构设计
   - 日期解析、信息分流器（COMMON_QUESTION、PROJECT_QUESTION、PROJECT_INFO、COMMON_INFO）
   - 问题：对话历史会产生干扰，需要用单独 prompt 重写问题

3. 标签应该包括：chatbot、项目管理、向量存储、LangChain、对话系统等

4. 关联笔记：查看提供的笔记列表，主要是关于 MySQL、Clickhouse、数据库相关的技术笔记，与这个 chatbot 项目没有直接关联

让我生成索引：## 标签
chatbot, 项目管理, 向量存储, LangChain, 对话系统, FAISS

## 摘要
记录使用 LangChain 和 FAISS 向量存储构建项目管理 chatbot 的实现过程，包括聊天记录存储和检索方案。20240131 更新引入日期解析和对话分流器，解决对话历史干扰问题。

## 关键概念
- 向量存储记忆：使用 FAISS 存储问答记录，每个问答作为独立 Document
- 对话分流器：区分 COMMON_QUESTION、PROJECT_QUESTION、PROJECT_INFO、COMMON_INFO 四种对话类型
- 日期解析：将模糊日期（如"明天"）转换为具体日期
- 问题重写：用独立 prompt 将当前问题重写成不依赖对话历史的完整问题

## 关联笔记
无
