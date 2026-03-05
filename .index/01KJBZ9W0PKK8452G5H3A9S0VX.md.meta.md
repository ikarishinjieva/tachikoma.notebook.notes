---
note: 01KJBZ9W0PKK8452G5H3A9S0VX.md
title: 20240613 - 阅读论文: FLAR: Active Retrieval Augmented Generation
indexed_at: 2026-03-05T10:06:00.754567+00:00
---

## 摘要
FLAR 提出在 LLM 推理过程中主动触发检索以完善答案的两种方法。FLAREinstruct 让模型直接生成检索指令，FLAREdirect 在 token 置信度低于阈值时自动检索替换不确定句子。

## 关键概念
- FLAREinstruct: LLM 推理时直接生成检索鼓励指令触发进一步检索
- FLAREdirect: 当生成 token 的置信概率小于阈值时，对所在句子进行主动检索替换
- 主动检索 (Active Retrieval): 在推理过程中动态决定何时进行额外检索而非预先检索
- 置信度阈值: 用于判断 LLM 生成内容不确定性、触发检索的概率阈值

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 流程增强综述笔记，在"基于规则判断召回必要性"部分引用了 FLAR 论文
- 01KJBZR62TY2F0CMS66NRZXDFS.md: Golden-Retriever 论文笔记，同属 RAG 增强方向，通过术语查询解释优化检索
- 01KJBZG9HE3MZ0312PBHKXC0M7.md: Speculative RAG 论文笔记，同属 RAG 优化方向，通过草稿机制降低上下文处理成本
