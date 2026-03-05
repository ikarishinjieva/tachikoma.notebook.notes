---
note: 01KJBZWMER6XKW0WW7KF0Y4HXM.md
title: 20250807 - 阅读论文*: The Entropy Mechanism of Reinforcement Learning for Reasoning Language Models
indexed_at: 2026-03-05T11:50:57.992512+00:00
---

## 标签
强化学习, 熵坍塌, 协方差正则化, 策略梯度, 推理模型, 梯度裁剪

## 摘要
论文揭示了 RL 训练早期策略熵急剧下降导致探索能力丧失的"熵坍塌"现象。提出 Clip-Cov 和 KL-Cov 两种协方差正则化方法，通过识别并限制高协方差词元的梯度更新来控制熵降。

## 关键概念
- 熵坍塌: RL 训练早期策略熵急剧降至接近零，导致模型丧失探索新推理路径的能力
- 协方差: 某个上下文和动作词元一起发生的概率，高协方差表示模型对该词元更"自信"
- Clip-Cov: 识别协方差极高的词元，在策略梯度计算中将其梯度裁剪掉使其不参与更新
- KL-Cov: 对协方差最高的 top-k 词元施加 KL 散度惩罚，限制策略更新幅度

## 关联笔记
- 01KJBZWQZ9BA47ET5VQBKGXEVE.md: 分析 GRPO loss 中熵的计算和处理方式，与本笔记的熵概念直接相关
- 01KJC008VJ7N9T3VR0Q5C4YPK8.md: 研究 RL 训练中的遗忘问题，同样涉及 KL 散度与策略更新的关系
