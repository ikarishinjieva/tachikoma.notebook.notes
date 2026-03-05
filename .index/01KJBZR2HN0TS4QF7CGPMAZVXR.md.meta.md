---
note: 01KJBZR2HN0TS4QF7CGPMAZVXR.md
title: 20250202 - 阅读论文*: LATE CHUNKING: CONTEXTUAL CHUNK EMBEDDINGS USING LONG-CONTEXT EMBEDDING MODELS
indexed_at: 2026-03-05T11:19:45.207044+00:00
---

## 标签
Late Chunking, 长上下文 Embedding, 池化操作, 语义理解, 向量表示, 词嵌入

## 摘要
提出 Late Chunking 方法：利用长上下文 embedding 模型生成包含语义信息的词嵌入，再对 chunk 内的词向量进行池化得到整体表示。该方法解决了传统 embedding 缺乏上下文理解的问题，且 LLM 无需特殊训练即可有效处理池化后的向量。

## 关键概念
- Late Chunking: 先获取词级 embedding 再池化得到 chunk 表示的方法
- 长上下文 Embedding 模型: 能够捕捉词与词之间关系及整体上下文的 embedding 模型
- 池化操作: 对 chunk 内词向量进行平均或求和，保留关键语义信息
- 上下文词嵌入: 包含词在整个文本中上下文信息的向量表示

## 关联笔记
- 01KJBZFPM6BX07HW9FQM0QZ670.md: 分析 dsRAG 项目的 chunking 创新技术，与本文的 Late Chunking 形成对比
- 01KJBZG9HE3MZ0312PBHKXC0M7.md: Speculative RAG 涉及文档召回后的处理策略，与 embedding 表示方法相关
