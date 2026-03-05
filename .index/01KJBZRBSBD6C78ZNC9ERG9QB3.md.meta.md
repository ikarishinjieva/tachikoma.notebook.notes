---
note: 01KJBZRBSBD6C78ZNC9ERG9QB3.md
title: 20250207 - 使用ms-swift对Qwen2-VL进行微调 [23] - 在基模型上增加原子数
indexed_at: 2026-03-05T11:26:31.912382+00:00
---

## 摘要
记录在已训练的基础原子+ring+branch 模型 checkpoint 上，通过两阶段训练扩展分子长度识别能力至 8-9。第一阶段使用长度 3-8 基础原子数据（含镜像和子段轮换增强），第二阶段加入 ring 原子数据，目标是提高长分子准确率。

## 关键概念
- ms-swift: 模型微调框架，用于 Qwen2-VL 的训练和推理
- 课程学习 (Curriculum Learning): 分步骤递进式训练策略，从简单到复杂
- 数据增强: 镜像增强、子段轮换增强、rotate 增强等扩展训练数据的方法
- checkpoint: 训练过程中的模型保存点，支持续训和评估
- Selfies 表示: 分子结构的字符串表示方法，用于训练数据生成

## 关联笔记
- 01KJBZR0H7TBJT1Y07GE1DHEBD.md: 直接前继，训练识别基础原子的基模型（训练方式 21）
- 01KJBZR66G465R50X6V0TXKB9K.md: 同系列前继，讨论 LR 对推理能力的影响及 resume_only_model 使用
- 01KJBZRQQVCZ5ZSZZ4W1F9974D.md: 包含完整的训练记录和 checkpoint 追踪，涵盖后续训练步骤
