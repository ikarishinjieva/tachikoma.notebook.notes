---
note: 01KJBZPTXXKTXVDR009ADNWMAK.md
title: 20250106 - 阅读论文*: Coconut: Training Large Language Models to Reason in a Continuous Latent Space
indexed_at: 2026-03-05T11:11:01.131183+00:00
---

## 摘要
Coconut 论文提出将 LLM 最后一层隐藏状态向量直接作为下一轮推理输入，替代传统词嵌入向量，以承载更丰富的推理信息。该方法面临隐藏状态向量不属于有效词嵌入空间的逻辑问题，需要模型同时处理两种不同的输入空间。

## 关键概念
- 隐藏状态向量: LLM 最后一层输出的高维向量，承载推理过程的中间表示
- 词嵌入向量: 将 token ID 映射到高维语义空间的向量，是标准 LLM 的输入形式
- 连续潜在空间推理: 用连续向量而非离散文字进行多轮推理的方法
- CoT(Chain of Thought): 思维链推理，将复杂问题分解为多步推理过程

## 关联笔记
- 01KJBZNKZMMHGYY4D6PE7BBQMW.md: 详细解释了 Token ID 与嵌入向量的关系，有助于理解 Coconut 用隐藏状态替代词嵌入的技术挑战
- 01KJBZTFWATGMK1VW5NRVDQKMP.md: 分析混元模型的 Transformer 架构，提供理解 LLM 内部结构的背景知识
- 01KJBZS9G27XHK76W77HVN48XA.md: 详解 Transformer 块结构和张量并行，帮助理解隐藏状态向量在模型中的生成过程
