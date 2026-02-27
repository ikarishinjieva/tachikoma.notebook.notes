---
note: 01KJBZ9W0PKK8452G5H3A9S0VX.md
title: 20240613 - 阅读论文: FLAR: Active Retrieval Augmented Generation
indexed_at: 2026-02-27T10:59:03.315326+00:00
---

用户要求我分析一篇关于 FLAR 论文的笔记，并生成索引元数据。让我仔细分析笔记内容：

笔记主题：FLAR (Active Retrieval Augmented Generation) - 主动检索增强生成

主要内容：
- 在 LLM 推理时主动进行检索来完善答案
- 两种触发检索的方式：
  - FLAREinstruct: 生成检索鼓励指令
  - FLAREdirect: 当 token 置信度低于阈值时触发检索

需要生成：
1. 标签（3-6 个）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

看其他笔记列表，大部分是关于 MySQL、Clickhouse、数据库、系统性能等主题的。有一篇笔记 "01KJBYDD09MGYR4GA0C4KB7P9F.md: 20210613 - TODO: Advanced Join Strategies for Large-Scale Distributed Computation" 可能与检索/分布式计算有一些关联，但关联度不高。其他笔记主要是数据库和系统层面的，与 LLM/检索增强生成关系不大。

让我生成索引：

标签：LLM, 检索增强生成，FLAR, 主动检索，推理优化

摘要：介绍 FLAR 主动检索增强生成方法，在 LLM 推理过程中主动触发检索以完善答案。

提出两种检索触发机制：FLAREinstruct 通过生成检索指令，FLAREdirect 基于 token 置信度阈值判断是否需要检索。

关键概念包括 FLAREinstruct 和 FLAREdirect 两种方法，以及 token 置信度阈值作为触发条件。关联笔记方面，现有数据库相关的笔记与这篇 LLM 检索主题关联度较低。
## 标签
LLM, 检索增强生成，FLAR, 主动检索，推理优化

## 摘要
介绍 FLAR 主动检索增强生成方法，在 LLM 推理过程中主动触发检索以完善答案。提出两种检索触发机制：FLAREinstruct 通过生成检索指令，FLAREdirect 基于 token 置信度阈值。

## 关键概念
- FLAREinstruct: LLM 推理时直接生成检索鼓励指令来触发检索
- FLAREdirect: 当 token 置信概率小于阈值时主动检索替换该句子
- 主动检索：在推理过程中根据需要主动发起检索而非被动等待

## 关联笔记
无
