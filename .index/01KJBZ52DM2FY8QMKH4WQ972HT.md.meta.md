---
note: 01KJBZ52DM2FY8QMKH4WQ972HT.md
title: 20240104 - 对langchain的ConversationEntityMemory的分析
indexed_at: 2026-03-05T09:26:54.740537+00:00
---

## 标签
langchain, ConversationEntityMemory, 对话记忆, 实体分析, LLM, 内存管理

## 摘要
分析 langchain 的 ConversationEntityMemory 组件工作机制。通过 LLM 对对话输入进行实体抽取和分析，支持按实体维度查询历史摘要，并将对话总结保存到对应实体中。

## 关键概念
- load_memory_variables: 在 chain.prep_inputs 中调用，用 LLM 分析输入并抽取实体，查询实体历史摘要
- save_context: 在 chain.prep_outputs 中调用，将对话输入输出保存到对应实体的总结中
- entity: 从对话中抽取的实体，作为记忆存储和检索的维度

## 关联笔记
- 01KJBYYMHN0C5CA4ARZ21B6AAN.md: 同属 langchain 对话记忆组件 (ConversationBufferMemory) 的使用测试
