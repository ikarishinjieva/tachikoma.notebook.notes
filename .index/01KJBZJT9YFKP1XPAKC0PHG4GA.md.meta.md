---
note: 01KJBZJT9YFKP1XPAKC0PHG4GA.md
title: 20241112 - 使用ms-swift对Qwen2-VL进行微调 [1]
indexed_at: 2026-03-05T10:57:06.061380+00:00
---

## 标签
Qwen2-VL, ms-swift, LoRA 微调，过拟合，环境配置，多模态模型

## 摘要
记录使用 ms-swift 框架对 Qwen2-VL-7B-Instruct 进行 LoRA 微调的初始实验，包含环境配置、训练命令和多组过拟合调优尝试。通过调整 weight_decay 和 lora_dropout 参数尝试缓解过拟合，但训练曲线无明显改善。

## 关键概念
- LoRA: 低秩适配器微调方法，减少可训练参数量
- ms-swift: ModelScope 提供的多模型微调框架
- weight_decay: 权重衰减正则化参数，用于缓解过拟合
- lora_dropout: LoRA 层的 dropout 率，控制过拟合程度
- Qwen2-VL: 阿里通义千问视觉语言模型

## 关联笔记
- 01KJBZRSYVSHF6VCBP86KAQ3RD.md: 第一阶段微调经验总结，引用本笔记中减少 LoRA 层数减轻过拟合的经验
- 01KJBZJWFDR591GB209N8PKJ1F.md: 系列 [3] 从本笔记 v0 版本出发，增加校验集比例进行调整
- 01KJBZN4KQV7NK5GRT46165QTR.md: 引用本笔记了解多模态大模型结构和微调方法
