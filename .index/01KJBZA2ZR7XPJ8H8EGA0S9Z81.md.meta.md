---
note: 01KJBZA2ZR7XPJ8H8EGA0S9Z81.md
title: 20240614 - 阅读论文*: Adaptive-RAG: Learning to Adapt Retrieval-Augmented Large Language Models through Question Complexity
indexed_at: 2026-02-27T11:00:21.923683+00:00
---

用户要求我分析一篇关于 Adaptive-RAG 论文的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记标题：关于 Adaptive-RAG 论文，涉及检索增强生成和大语言模型
2. 主要内容：
   - 通过问题复杂度分类器使用不同策略
   - 三种策略：无检索、单步检索、多步检索
   - 分类器训练可以自动标记，无需人工标记

3. 标签应该提取 3-6 个关键词：RAG、大语言模型、检索策略、问题分类、自动训练等

4. 摘要：2-3 句话概括核心内容

5. 关键概念：3-5 个核心概念及其解释

6. 关联笔记：查看提供的笔记列表，大部分是关于 MySQL、Clickhouse、系统调试等主题，与 RAG/LLM 不相关。有一个笔记 "01KJBYDD09MGYR4GA0C4KB7P9F.md: 20210613 - TODO: Advanced Join Strategies for Large-Scale Distributed Computation" 可能有一些关联（都涉及策略选择），但关联度很低。所以关联笔记应该是「无」。

让我生成输出：
## 标签
RAG, 大语言模型, 检索策略, 问题分类, 自动训练

## 摘要
Adaptive-RAG 通过问题复杂度分类器动态选择检索策略：简单问题直接生成、中等问题单步检索、复杂问题多步迭代检索。分类器可通过对比不同策略在样例问题上的效果自动训练，无需人工标注。

## 关键概念
- 无检索：直接使用 LLM 生成答案，适用于最简单问题
- 单步检索：检索相关文档后输入 LLM 生成答案，适用于中等复杂度问题
- 多步检索：迭代检索文档并生成中间答案，适用于最复杂问题
- 自动标注：通过对比不同策略效果自动确定训练数据标签

## 关联笔记
无
