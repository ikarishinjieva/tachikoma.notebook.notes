---
note: 01KJBZEQM62M6NKXGRYN8MRJFZ.md
title: 20240826 - 阅读论文: The Power of Scale for Parameter-Efficient Prompt Tuning
indexed_at: 2026-03-05T10:37:44.736970+00:00
---

## 摘要
介绍 Prompt Tuning 方法，通过冻结预训练模型参数、仅训练可学习 Prompt 向量实现高效微调。讨论了 Prompt 初始化策略、长度设计和预训练目标选择等关键设计决策，以及在大规模语言模型上的应用潜力。

## 关键概念
- Prompt Tuning: 冻结预训练模型参数，仅训练插入输入开头的可学习 Prompt 向量的微调方法
- Soft Prompt: 不来自词表、拥有独立可学习嵌入向量的特殊 token 序列
- 参数效率: 只训练少量参数，降低存储和计算成本，避免过拟合
- Prompt 初始化: 可通过随机初始化、词表采样或类别标签嵌入向量初始化 Prompt
- 预训练目标: 自回归语言建模比掩码语言建模更适合 Prompt Tuning

## 关联笔记
- 01KJBZE9R01Q1V5W8595T0XR1R.md: 笔记中明确引用的 OpenPrompt 框架说明，讲解 prompt-learning 和 soft prompt 技术
