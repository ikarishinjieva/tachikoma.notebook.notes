---
note: 01KJBZYAS3THKM4CYDK96M7AQV.md
title: 20250819 - 阅读论文*: GLM-4.5: Agentic, Reasoning, and Coding (ARC) Foundation Models
indexed_at: 2026-03-05T11:54:16.323134+00:00
---

## 摘要
GLM-4.5 论文后训练部分的核心策略：精细化数据筛选（保留高难度提示 + 多响应缩放）和单阶段 64K 长文本 RL 训练。采用迭代式自蒸馏流程（SFT→RL→生成新轨迹→筛选→新 SFT）持续提升推理能力。

## 关键概念
- 响应级别缩放：对每个高价值提示生成四个不同回答，放大优质样本训练价值
- 长文本能力遗忘：多阶段 RL 会导致模型忘掉 SFT 阶段学到的长文本生成能力
- 迭代式自蒸馏：通过多轮 SFT-RL 循环，让模型生成更高质量的推理轨迹
- 领域定制 RL 损失：使用专家验证的多项选择题进行科学领域 RL 训练

## 关联笔记
- 01KJC008VJ7N9T3VR0Q5C4YPK8.md: 研究 RL 相比 SFT 遗忘更少的机制，与长文本 RL 遗忘问题相关
- 01KJBZWMER6XKW0WW7KF0Y4HXM.md: 分析 RL 训练中的熵坍塌现象及控制方法
- 01KJBZWP1711J31GDNMNTAQW01.md: 高熵词元在 LLM 推理 RL 中的作用机制
