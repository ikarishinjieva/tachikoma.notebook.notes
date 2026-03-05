---
note: 01KJBZ9R4MN3T2EN6DV5JDPR7F.md
title: 20240610 - 阅读论文: Active Prompt
indexed_at: 2026-03-05T10:02:39.552250+00:00
---

## 摘要
Active Prompt 通过不确定性度量（分歧度、熵、方差、自信度）选择最不确定的问题供人工标注，优化 CoT 的 few-shot 示例选择。实验表明分歧度和熵效果最佳，自信度因 LLM 过度自信问题效果不佳。

## 关键概念
- 思维链 (CoT): 使用 few-shot 提示让 LLM 展示逻辑推导过程
- 分歧度 (Disagreement): 基于模型多次预测结果的差异性衡量不确定性
- 熵 (Entropy): 信息论概念，衡量模型预测概率分布的不确定性
- 方差 (Variance): 衡量多次预测结果的数值离散程度
- 主动学习: 选择不确定度最高的样本进行人工标注以提升模型性能

## 关联笔记
- 01KJBZRE950XFNR96B42N1J08E.md: 同样讨论了使用反射标记评估模型不确定性的方法
- 01KJBZYC8BKTFV07JJNAQNS91T.md: 探讨思维链忠实性，与 CoT 推理机制相关
- 01KJBZR3H94N01JGGGAYA36GP2.md: 研究不依赖提示的 CoT 推理，与 CoT 优化方向相关
