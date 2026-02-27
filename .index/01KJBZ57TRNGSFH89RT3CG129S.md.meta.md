---
note: 01KJBZ57TRNGSFH89RT3CG129S.md
title: 20240203 - 阅读论文: Query Rewriting for Retrieval-Augmented Large Language
indexed_at: 2026-02-27T10:43:04.760971+00:00
---

用户希望我分析这篇关于论文阅读的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于"Query Rewriting for Retrieval-Augmented Large Language"论文的笔记
2. 核心内容是关于RAG（检索增强生成）中的查询重写技术
3. 主要方法：使用伪数据预热 + 强化学习来训练重写器
4. 框架：Rewrite-Retrieve-Read

需要生成的内容：
1. 标签：3-6个关键词
2. 摘要：2-3句话
3. 关键概念：3-5个核心概念
4. 关联笔记：从给定的笔记列表中找0-3篇相关的

关于关联笔记，我需要看是否有与RAG、查询重写、语言模型、强化学习等相关的笔记。浏览列表：
- 大部分是关于MySQL、Clickhouse、数据库相关的笔记
- 没有明显与RAG、查询重写、LLM相关的笔记
- 01KJBYDD09MGYR4GA0C4KB7P9F.md: TODO: Advanced Join Strategies... 这个可能有一点点相关（都是检索相关），但实际是数据库join策略，不太相关
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 关于Clickhouse的，不相关

所以关联笔记应该是「无」

让我生成元数据：

标签：RAG, 查询重写，强化学习，伪数据，语言模型，检索增强


摘要：这篇论文介绍了Rewrite-Retrieve-Read框架，通过查询重写来改进检索增强的语言模型性能。采用伪数据预热和强化学习相结合的方法训练可训练的重写器，无需标注数据即可优化查询生成。

关键概念：
1. Rewrite-Retrieve-Read: 三阶段框架，先重写查询，再检索，最后阅读生成
2. 伪数据预热：用LLM生成伪标签来初始化重写器
3. 强化学习：用PPO算法根据阅读器反馈优化重写器
4. 可训练重写器：基于T5-large的小型语言模型，负责查询重写

关联笔记：无
## 标签
RAG, 查询重写，强化学习，伪数据，语言模型，检索增强

## 摘要
本文介绍了一种通过查询重写改进检索增强语言模型（RAG）的方法，采用 Rewrite-Retrieve-Read 三阶段框架。使用伪数据预热结合强化学习（PPO）训练可训练重写器，解决无标注数据情况下的模型优化问题。

## 关键概念
- Rewrite-Retrieve-Read: 三阶段流程，先重写查询、再检索、最后由语言模型阅读生成答案
- 伪数据预热：用 LLM 生成伪标签构建训练集，对重写器进行初始微调
- 强化学习训练：通过 PPO 算法，以阅读器反馈为奖励进一步优化重写器
- 可训练重写器：基于 T5-large 的小型语言模型，独立于黑盒 LLM 进行训练

## 关联笔记
无
