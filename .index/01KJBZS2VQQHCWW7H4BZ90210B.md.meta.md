---
note: 01KJBZS2VQQHCWW7H4BZ90210B.md
title: 20250318 - 阅读论文*: Logic-RL: Unleashing LLM Reasoning with Rule-Based Reinforcement Learning
indexed_at: 2026-03-05T11:34:40.927631+00:00
---

## 标签
强化学习，LLM 推理，Logic-RL，规则奖励，泛化能力，REINFORCE++

## 摘要
论文提出 Logic-RL 框架，通过基于规则的强化学习在逻辑谜题上训练 7B 模型，在 AIME 和 AMC 基准测试中分别提升 125% 和 38%。关键发现包括 RL 比 SFT 泛化能力更强、冷启动非必需、语言混合阻碍推理、增加思考 tokens 有帮助。

## 关键概念
- Logic-RL: 基于规则的强化学习框架，使用逻辑谜题训练 LLM 推理能力
- 基于规则的奖励: 包含系统提示、格式奖励、答案奖励的奖励系统，难以被模型破解
- KL 散度惩罚: 将 RL 模型与 SFT 模型的 KL 散度作为惩罚项，防止策略过度偏离
- REINFORCE++: 修改后的 RL 算法，使用 KL 损失和无偏估计器
- OOD 泛化: 模型对训练分布外任务（如 8 人谜题、AIME/AMC）的泛化能力

## 关联笔记
- 01KJBZYAS3THKM4CYDK96M7AQV.md: 讨论 SFT→RL→生成新轨迹的迭代训练流程和单阶段长文本 RL 策略
- 01KJC008VJ7N9T3VR0Q5C4YPK8.md: 研究 RL 相比 SFT 遗忘更少的原因，与 KL 散度和在线学习机制相关
- 01KJBZY0MAYHH592A9NG68XWV3.md: 探索小模型上使用 GRPO 进行推理训练的方法和奖励设计
