---
note: 01KJBZ9SH4B3BFY5NX5K28FXZ3.md
title: 20240610 - 阅读论文: ReMoDiffuse: Retrieval-Augmented Motion Diffusion Model
indexed_at: 2026-03-05T10:04:13.238586+00:00
---

## 摘要
ReMoDiffuse 针对检索增强动作生成的三大挑战（相似性评估、知识选择性吸收、尺度敏感）提出了解决方案。核心创新包括混合检索（结合语义与运动学相似性）、语义调制 Transformer（过滤调整检索信息）和条件混合技术（动态平衡多条件影响）。

## 关键概念
- 混合检索 (Hybrid Retrieval): 结合语义相似性和运动学相似性（如动作序列长度）进行检索
- 语义调制 Transformer (Semantics-Modulated Transformer): 根据文本与检索样本的语义差异过滤和调整检索信息
- 条件混合 (Condition Mixture): 学习不同权重参数动态调整文本、检索样本和噪声信号在生成过程中的影响
- 分类器自由引导 (Classifier-Free Guidance): 扩散模型中条件信息的尺度控制技术
- 检索增强动作生成 (Retrieval-Augmented Motion Generation): 利用检索样本增强动作扩散模型的生成质量

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文，提供了检索增强生成的分类框架和技术背景
- 01KJBZTFWATGMK1VW5NRVDQKMP.md: 介绍扩散模型在多模态学习中的应用，与 ReMoDiffuse 的扩散模型基础相关
