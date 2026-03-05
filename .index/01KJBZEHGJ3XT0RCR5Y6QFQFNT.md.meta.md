---
note: 01KJBZEHGJ3XT0RCR5Y6QFQFNT.md
title: 20240820 - 阅读论文*: Guiding Large Language Models via Directional Stimulus Prompting
indexed_at: 2026-03-05T10:36:17.881141+00:00
---

## 摘要
论文提出"方向性指引"概念，通过在提示词中提供关键词列表、对话行为、触发词等引导信号来增强 LLM 输出质量。训练方法采用 SFT+RL 两阶段策略：先用监督微调让模型学会生成指引，再用强化学习探索更优的提示空间。

## 关键概念
- 方向性指引 (Directional Stimulus): 提示词中的引导信号（如关键词、对话行为、触发词），用于引导 LLM 生成符合预期的高质量输出
- 策略模型 (Policy Model): 小型可调语言模型（如 T5），用于学习根据输入生成方向性指引
- ROUGE 奖励: 用 ROUGE 指标衡量生成摘要与参考摘要的相似度，作为 RL 的奖励信号
- SFT+RL 两阶段训练: 先用 SFT 提供良好起点，再用 RL 突破训练数据空间限制探索最优策略

## 关联笔记
- 01KJC008VJ7N9T3VR0Q5C4YPK8.md: 同样研究 SFT 与 RL 的关系，解释为何 RL 比 SFT 遗忘更少
- 01KJBZYAS3THKM4CYDK96M7AQV.md: 记录 GLM-4.5 论文的 SFT→RL 迭代训练流程，与本笔记训练方法相似
