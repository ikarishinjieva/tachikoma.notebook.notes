---
note: 01KJBZJG2DN96WB9H43JKKN8MY.md
title: 20241107 - 训练模型(化学分子式识别) - 尝试使用CoT
indexed_at: 2026-03-05T10:55:00.019179+00:00
---

## 摘要
记录使用 CoT（思维链）方法训练 Qwen2-VL 模型识别化学分子式并生成 SMILES 表达式的实验。对比 2000 和 3000 数据量的训练效果，发现过拟合现象且拐点出现在 warmup 结束时。

## 关键概念
- CoT (Chain of Thought): 让模型输出逐步分析图片构建 SMILES 表达式的推理过程
- SMILES: 分子结构的线性字符串表示法，用于描述化学分子式
- 过拟合: 训练集 loss 持续下降但验证集 loss 在 warmup 结束后开始上升
- Warmup: 训练初期学习率逐步上升的阶段，影响模型收敛行为

## 关联笔记
- 01KJBZRSYVSHF6VCBP86KAQ3RD.md: 总结分子式识别微调经验，提及 CoT 流程和多任务训练的尝试
- 01KJBZNVMJEEW6VS461MEX1YH0.md: 2024 年 12 月阶段性移交文档，延续本笔记的训练方向
