---
note: 01KJBZQFTGNMPCZZCMKYAXQR6K.md
title: 20250115 - 使用ms-swift对Qwen2-VL进行微调 [19] - 调优长度7的效果
indexed_at: 2026-03-05T11:13:04.843145+00:00
---

## 标签
Qwen2-VL, 分子结构识别, 课程学习, 数据增强, 微调实验, SELFIES

## 摘要
记录使用 ms-swift 对 Qwen2-VL 进行分子结构识别微调的长度 7 调优实验，对比增加长度 6 数据基线与简化训练步骤两种策略。实验显示长度 6 准确率可达 86-89%，但长度 7 准确率仅 50-65%，主要错误类型为原子序列错误和环结构识别错误。

## 关键概念
- 课程学习 (Curriculum Learning): 从短长度分子逐步训练到长长度分子的递进式训练策略
- SELFIES: 一种鲁棒性更好的分子表示方法，支持镜像反转和随机化数据增强
- CoT (Chain of Thought): 在提示词中指定原子数和推理流程，引导模型逐步识别分子结构
- 数据增强: 通过图形变换、SMILES 随机化、镜像 SELFIES 等方式扩充训练数据集

## 关联笔记
- 01KJBZQHK4TKYKDV1XKS00J7CM.md: 同一实验系列的第 2 篇，继续调优长度 7 的基线实验
- 01KJBZQT2DD1YNRJ2J49PP9MN0.md: 同一实验系列的第 3 篇，增加 ring 规范数据后的调优
- 01KJBZRSYVSHF6VCBP86KAQ3RD.md: 第一阶段微调经验总结，引用了本笔记的数据增强和课程学习经验
