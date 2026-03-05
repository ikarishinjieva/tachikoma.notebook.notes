---
note: 01KJBZZHDRJBS1FEX3M60AZ305.md
title: 20250903 - 阅读论文*: DoRA: Weight-Decomposed Low-Rank Adaptation
indexed_at: 2026-03-05T11:55:02.352795+00:00
---

## 摘要
DoRA 通过将权重分解为大小 (Magnitude) 和方向 (Direction) 两个独立组件，在反向传播时实现梯度的智能分流，解决了 LoRA 中大小与方向更新耦合的问题。相比 LoRA，DoRA 更接近全量微调的更新模式，在实际 SQL 优化 GRPO 训练中收敛更快。

## 关键概念
- DoRA: 权重分解低秩适应，将权重矩阵分解为大小向量和方向单位矩阵分别训练
- LoRA: 低秩适应微调方法，在冻结权重旁添加可训练的低秩补丁矩阵
- 梯度分流: DoRA 反向传播时将总梯度分解为投影分量 (更新大小) 和垂直分量 (更新方向)
- 全量微调 (FT): 模型所有参数参与训练，大小和方向更新呈负相关解耦模式
- 大小 - 方向解耦: DoRA 核心创新，使模型能独立精细调整权重的大小或方向

## 关联笔记
- 01KJBZZJ92R5GJ0VH1J4W3X93K.md: 笔记中提到的实际应用验证场景，DoRA 在此 GRPO 训练中比 LoRA 收敛更快
- 01KJBZZJ34XCYFH4C8WXDKJAMV.md: 同一天阅读的另一篇 LoRA 改进论文，rsLoRA 通过缩放因子修正解决高秩训练不稳定问题
