---
note: 01KJBZJVK3NEV6BHJ6K77YKYC5.md
title: 20241114 - 使用ms-swift对Qwen2-VL进行微调 [2] - 使用SELFIES替代SMILES
indexed_at: 2026-03-05T10:58:21.577208+00:00
---

## 摘要
使用脚本生成基于 SMILES 的 COT（思维链）数据，并发现原始数据中存在 SMILES 标记错误。这是 Qwen2-VL 微调系列的第 2 篇，探索用 SELFIES 替代 SMILES 进行分子结构识别的微调方法。

## 关键概念
- SELFIES: 自引用嵌入式字符串，一种比 SMILES 更鲁棒的分子表征方法
- SMILES: 简化分子线性输入系统，用于表示分子结构的字符串记号
- COT (Chain of Thought): 思维链，让模型输出推理过程以提升准确性
- Qwen2-VL: 阿里云开源的多模态大语言模型，支持任意分辨率图像输入
- ms-swift: 魔搭社区提供的模型微调框架

## 关联笔记
- 01KJBZJ2B4WEX7J5JS6W6BXGYH.md: Qwen2-VL 论文阅读笔记，为微调提供理论基础
- 01KJBZJG2DN96WB9H43JKKN8MY.md: 尝试使用 CoT 进行训练的前序笔记
- 01KJBZP6X3BNGMNB5KJYX0X60Q.md: 同系列第 13 篇，探索递增式训练方法
