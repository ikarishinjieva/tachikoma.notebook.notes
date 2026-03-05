---
note: 01KJC008VJ7N9T3VR0Q5C4YPK8.md
title: 20250915 - 阅读论文*: RL’S RAZOR: WHY ONLINE REINFORCEMENT LEARNING FORGETS LESS
indexed_at: 2026-03-05T11:58:14.856925+00:00
---

## 摘要
论文揭示 RL 比 SFT 遗忘更少的核心原因：在线 RL 训练产生的分布与原模型 KL 散度更低。通过构建"神谕 SFT"数据集和对比四种算法（在线/离线 × 正/负样本），证明"在线策略"是遗忘差异的关键因素。

## 关键概念
- KL 散度: 衡量新知识与现有模型分布偏离程度的指标，偏离越低遗忘越少
- 在线强化学习: 使用模型自身输出进行训练的学习方式，产生更低的 KL 散度
- 神谕 SFT: 人工构建的低 KL 散度数据集，可复现 RL 的低遗忘效果
- 前向 KL 散度: 比其他指标（权重变化、激活值变化等）更能解释遗忘现象

## 关联笔记
- 01KJBZYAS3THKM4CYDK96M7AQV.md: 讨论多阶段 RL 训练导致的"长文本能力遗忘"现象，与本研究互补
- 01KJBZY9ADDRRK6F59FBPBYDT1.md: 记录 SFT→DPO→GRPO 的训练流程，涉及 SFT 与 RL 的对比
