---
note: 01KJBZRQ6GE4B51ZFRHXA7TN44.md
title: 20250303 - 阅读论文*: SFT Memorizes,RL Generalizes:A Comparative Study of Foundation Model Post-training
indexed_at: 2026-03-05T11:30:53.643473+00:00
---

## 摘要
论文对比 SFT 与 RL 两种后训练方法，发现 RL 在规则泛化和视觉泛化任务上显著优于 SFT，而 SFT 更依赖训练数据记忆。研究通过 GeneralPoints 卡牌游戏和 V-IRL 导航任务两个多模态评估环境验证结论。

## 关键概念
- SFT（监督微调）: 通过监督学习微调模型，在同分布任务表现好但依赖数据记忆
- RL（强化学习）: 通过奖励函数优化策略，在 OOD 场景下泛化能力更强
- OOD 泛化: 模型在未见过分布外的数据上的泛化表现
- 多步 RL 框架: 结合验证器实现迭代式错误修正的强化学习框架
- 验证迭代: 推理时增加验证次数可进一步提升 RL 泛化能力

## 关联笔记
- 01KJC008VJ7N9T3VR0Q5C4YPK8.md: 同为主题研究 SFT 与 RL 差异，分析 RL 遗忘更少的原因
- 01KJC06PG4RM9T7WFDEN5HPZPN.md: 讨论强化学习与模仿学习在机器人任务中的泛化应用
