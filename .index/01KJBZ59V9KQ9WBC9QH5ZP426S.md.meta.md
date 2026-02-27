---
note: 01KJBZ59V9KQ9WBC9QH5ZP426S.md
title: 20240204 - 阅读论文: REPLUG: Retrieval-Augmented Black-Box Language Models
indexed_at: 2026-02-27T10:43:09.600587+00:00
---

用户要求我分析这篇关于 REPLUG 论文的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题是关于 REPLUG 论文的阅读笔记
2. 内容涉及检索增强语言模型、REPLUG 的工作原理、代码实现等
3. 其他笔记列表主要是关于数据库、MySQL、Clickhouse 等技术，与这篇 AI/LLM 主题的笔记没有明显关联

我需要生成：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个核心概念
4. 关联笔记：0-3 篇（看起来没有明显关联的）

让我整理输出。## 标签
REPLUG, 检索增强, 大语言模型, 论文阅读, logits processor, invocation layer

## 摘要
本笔记记录了 REPLUG 论文的核心内容，即通过检索模型增强黑盒语言模型性能的方法。重点整理了 REPLUG 的输入重构流程和代码实现，特别是 REPLUGLogitsProcessor 如何通过文档相似度权重修订生成概率。

## 关键概念
- REPLUG: 检索增强的黑盒语言模型框架，通过引入检索文档改进 LLM 预测
- 输入重构: 将检索文档与输入上下文拼接，并行传递给语言模型进行集成
- REPLUGLogitsProcessor: 根据文档相似度得分对 logits 进行加权修订的处理器
- Invocation Layer: 大模型最外层，组织 tokenizer + generate + decode 的整体流程

## 关联笔记
无
