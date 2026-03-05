---
note: 01KJBZQCSH2WSBPRYJQFMYM1C3.md
title: 20250111 - 使用ms-swift对Qwen2-VL进行微调 [18] - 使用相似COT流程进行两种互补任务
indexed_at: 2026-03-05T11:12:18.725221+00:00
---

## 摘要
记录使用相似 CoT 流程进行分子识别和修正两种互补任务的微调实验，对比一阶段和两阶段评估方案。多 epoch 训练结果显示 atom_seq 错误仍是主要问题，两阶段评估未带来明显提升。

## 关键概念
- SELFIES: 分子结构的字符串表示方法，比 SMILES 更鲁棒
- CoT 流程: Chain of Thought，引导模型逐步推理的思维链提示方法
- 一阶段评估: 直接输出最终修正结果的评估方式
- 两阶段评估: 先识别 selfies 再进行修正的分步评估方式
- atom_seq 错误: 原子序列识别错误，是模型的主要错误类型

## 关联笔记
- 01KJBZQ6DWHS6EB67JVEPB2H98.md: 前继笔记，探索增强 COT 流程的方案
- 01KJBZQGVRD2RTTJR6BNFDYD35.md: 同系列笔记，继续使用数据增强方法改进
- 01KJBZRSYVSHF6VCBP86KAQ3RD.md: 总结性笔记，引用本实验的核心经验
