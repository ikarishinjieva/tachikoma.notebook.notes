---
note: 01KJBZS20MPESNN97X6XPEZBV0.md
title: 20250318 - 阅读论文*: Chain of Draft: Thinking Faster by Writing Less
indexed_at: 2026-02-27T11:35:45.599142+00:00
---

用户要求我分析一篇关于"Chain of Draft (CoD)"论文的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 主要思想：让 CoT（Chain of Thought）只输出关键词句，每步输出 5 个单词左右，控制思考方向但不限制发散
2. 有详细示例对比 CoT 和 CoD 的输出
3. 分析了 CoD 提示词的关键要素

需要生成：
- 标签（3-6 个）
- 摘要（2-3 句话）
- 关键概念（3-5 个）
- 关联笔记（0-3 篇，从给定列表中选择）

关于关联笔记，我需要看是否有相关的。这篇笔记是关于 LLM 推理优化（CoD vs CoT），而给定的笔记列表主要是关于 MySQL、Clickhouse、系统性能等，看起来没有直接相关的。最接近的可能是"20210613 - TODO: Advanced Join Strategies for Large-Scale Distributed Computation"这个也是关于优化策略的论文笔记，但主题不同。所以关联笔记应该是"无"。

让我生成输出：

标签：大语言模型、推理优化、Chain of Draft、提示工程、token 效率、思维链

摘要：介绍 Chain of Draft(CoD) 方法，通过让模型每步只输出 5 个单词以内的关键词句来优化推理过程。

相比传统 CoT，这种方法在保持推理质量的同时显著降低 token 消耗和延迟。

关键概念包括 Chain of Draft 本身、与传统 Chain of Thought 的对比、5 词限制的设计、最小化草稿策略，以及分隔符在答案提取中的作用。

关联笔记方面，现有列表中缺乏直接相关的内容。
## 标签
大语言模型、推理优化、Chain of Draft、提示工程、Token 效率、思维链

## 摘要
介绍 Chain of Draft(CoD) 方法，通过让模型每步推理只输出 5 个单词以内的关键词句来优化推理过程。相比传统 CoT，CoD 在保持推理质量的同时显著降低 token 消耗和延迟，并提供了清晰的提示词编写指南。

## 关键概念
- Chain of Draft (CoD): 一种优化的推理方法，每步只保留 5 词以内的核心草稿
- Chain of Thought (CoT): 传统的逐步推理方法，输出完整的思考过程
- 最小草稿原则: 每步推理只保留最关键信息，减少冗余
- 分隔符提取: 使用 #### 分隔符标记最终答案便于提取

## 关联笔记
无
