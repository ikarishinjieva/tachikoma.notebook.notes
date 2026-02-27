---
note: 01KJBZETD71Z66AHRQNQ181WNG.md
title: 20240826 - 阅读论文: AUTOMATIC CHAIN OF THOUGHT PROMPTING IN LARGE LANGUAGE MODELS
indexed_at: 2026-02-27T11:14:06.002806+00:00
---

用户要求我分析这篇关于 Auto-CoT 论文的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**：这是一篇关于大语言模型（LLM）中 Chain of Thought (CoT) 提示技术的论文笔记，具体是 Auto-CoT 方法。

2. **核心内容**：
   - Auto-CoT 是一种自动构建 CoT 演示的方法
   - 包含两个阶段：问题聚类和演示采样
   - 使用 Sentence-BERT 进行问题编码，k-means 聚类
   - 从每个聚类中选择代表性问题，用 Zero-Shot-CoT 生成推理链
   - 笔记还指出了算法的缺陷：聚类空间不等于推理深度

3. **标签**：应该是关于 LLM、CoT、Prompt Engineering、机器学习等

4. **关键概念**：Auto-CoT、Chain of Thought、Sentence-BERT、Zero-Shot-CoT、k-means 聚类等

5. **关联笔记**：查看提供的笔记列表，大部分是关于 MySQL、Clickhouse、数据库、系统性能等方面的笔记。其中有一个可能相关：
   - 01KJBYDD09MGYR4GA0C4KB7P9F.md: 20210613 - TODO: Advanced Join Strategies for Large-Scale Distributed Computation - 这个看起来是关于分布式计算的，但与 LLM/CoT 关系不大
   
其他笔记都是数据库和系统相关的，与这篇 LLM 论文笔记没有直接关联。

现在生成输出格式。## 标签
大语言模型, 思维链, Prompt 工程, Auto-CoT, 聚类算法, 论文笔记

## 摘要
笔记记录了 Auto-CoT 论文的核心方法：通过问题聚类 + 演示采样两阶段自动生成 CoT 样例。同时指出算法缺陷：Sentence-BERT 聚类空间与推理深度无必然关联。

## 关键概念
- Auto-CoT: 自动构建思维链演示的方法，无需人工编写样例
- Chain of Thought (CoT): 让大模型逐步推理的提示技术
- Sentence-BERT: 用于将问题编码为向量表示的嵌入模型
- Zero-Shot-CoT: 通过"Let's think step by step"触发模型推理
- 问题聚类: 使用 k-means 将相似问题分组以确保演示多样性

## 关联笔记
无
