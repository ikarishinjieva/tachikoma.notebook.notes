---
note: 01KJBZQYYB3G1G5MW2TJ5MPTXW.md
title: 20250131 - 阅读论文: CPO: Chain of Preference Optimization: Improving Chain-of-Thought Reasoning in LLMs
indexed_at: 2026-02-27T11:28:27.669868+00:00
---

用户希望我分析这篇关于 CPO (Chain of Preference Optimization) 论文的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20250131 - 阅读论文：CPO: Chain of Preference Optimization: Improving Chain-of-Thought Reasoning in LLMs

2. 笔记内容：主要是关于 CPO 方法的讨论，包括：
   - CPO 通过 DPO 训练让模型在 CoT 推理时选择更好的步骤
   - 训练数据来自 ToT (Tree of Thought) 搜索过程
   - 学习 ToT 的偏好信息
   - 链级优化，与 FPO 的区别

3. 其他笔记列表：看起来都是关于数据库、MySQL、Clickhouse、系统性能等方面的笔记，与 LLM/CPO 这个主题没有明显关联。

现在我需要生成：
- 标签：3-6 个关键词
- 摘要：2-3 句话
- 关键概念：3-5 个核心概念
- 关联笔记：0-3 篇（看起来没有关联的）

让我生成这些内容。
## 标签
CPO, DPO, 大语言模型, 链式思维, 偏好优化, ToT

## 摘要
CPO 方法利用 ToT 搜索过程中产生的偏好信息，通过 DPO 算法训练 LLM，使其在 CoT 推理过程中学习选择更优的推理步骤。与 FPO 不同，CPO 在链级进行优化，避免了梯度消失问题，更有效提升推理能力。

## 关键概念
- CPO (Chain of Preference Optimization): 通过 DPO 训练让模型学习 CoT 推理中每一步的偏好选择
- DPO (Direct Preference Optimization): 直接偏好优化算法，用于训练模型学习偏好
- ToT (Tree of Thoughts): 树状思维搜索方法，生成多候选推理路径并提供偏好数据
- FPO (Full-path Preference Optimization): 全路径偏好优化，仅关注最终答案而非链级优化
- LCP 梯度消失: FPO 方法容易出现的问题，CPO 通过链级优化避免此问题

## 关联笔记
无
