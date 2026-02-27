---
note: 01KJBZA2394KZG8T9YYWXT1CXS.md
title: 20240614 - 阅读论文*: Retrieve only when it needs: Adaptive retrieval augmentation for hallucination mitigation in large lan
indexed_at: 2026-02-27T11:00:04.848621+00:00
---

用户要求我分析这篇关于论文阅读的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：关于一篇论文"Retrieve only when it needs: Adaptive retrieval augmentation for hallucination mitigation in large lan"的阅读笔记，主要讲述了一种叫 Rowen 的方法，用于识别 LLM 产生的幻觉。

2. **核心内容**：
   - Rowen 方法通过检测相同查询在不同语言之间的不一致性来识别 LLM 幻觉
   - 解释了为什么多语言不一致性可能暗示幻觉（训练数据偏差、模型结构、翻译信息损失等）
   - 但也指出幻觉并不总是伴随多语言不一致性

3. **标签**：需要 3-6 个关键词，如：LLM、幻觉检测、跨语言、Rowen、检索增强

4. **关联笔记**：查看其他笔记列表，大部分是关于 MySQL、Clickhouse、系统性能等技术笔记，与这篇 LLM 幻觉检测的笔记关联度较低。但有一个可能相关：
   - 01KJBYDD09MGYR4GA0C4KB7P9F.md: 20210613 - TODO: Advanced Join Strategies for Large-Scale Distributed Computation - 这个也是关于论文/学习的，但主题是 Join 策略，关联度不高
   
   实际上这些笔记都是数据库、系统性能相关的，与 LLM 幻觉检测没有直接关联，所以关联笔记应该写「无」。

现在生成输出。
## 标签
LLM, 幻觉检测, 跨语言一致性, Rowen, 检索增强

## 摘要
介绍 Rowen 方法，通过检测相同查询在不同语言间的回答不一致性来识别 LLM 幻觉。分析了多语言幻觉差异的成因，包括训练数据偏差、模型结构、翻译信息损失等因素。

## 关键概念
- Rowen: 利用跨语言回答不一致性来识别 LLM 幻觉的方法
- 幻觉（Hallucination）: LLM 生成的与事实不符或错误的信息
- 跨语言一致性: 相同语义的查询在不同语言下应得到一致的回答

## 关联笔记
无
