---
note: 01KJBZJSM5YEKX18QPBPFMAYNX.md
title: 20241112 - 评估过拟合的原因
indexed_at: 2026-03-05T10:56:31.485977+00:00
---

## 摘要
分析 Qwen2-VL 模型在 img2smiles 任务中的过拟合现象，对比 checkpoint-7500 和 checkpoint-500 两个模型的输出效果。发现训练数据和测试数据均能正常输出 SMILES 表达式，并未出现 eval 阶段输出空值的情况，怀疑评估阶段数据本身存在问题。

## 关键概念
- 过拟合: 模型在训练集上表现良好但泛化能力不足的现象
- checkpoint: 训练过程中保存的模型权重快照，用于推理或续训
- SMILES: 分子结构的字符串表示法，用于化学分子式编码
- CoT (Chain of Thought): 思维链提示，让模型输出分步推理过程
- evaluate: 模型评估阶段，用于验证训练效果和选择最佳模型

## 关联笔记
- 01KJBZJMH94H8E9GBJBFTFCW58.md: 前序笔记，尝试查看 transformers 的 evaluate 输出结果以分析过拟合
- 01KJBZJT9YFKP1XPAKC0PHG4GA.md: 同日进行的 Qwen2-VL 微调首篇实验，记录减少 lora 层数减轻过拟合
- 01KJBZRSYVSHF6VCBP86KAQ3RD.md: Qwen2-VL 微调系列总结，包含过拟合问题的经验归纳
