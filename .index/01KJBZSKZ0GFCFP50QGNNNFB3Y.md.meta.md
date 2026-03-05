---
note: 01KJBZSKZ0GFCFP50QGNNNFB3Y.md
title: 20250506 - 阅读论文*: PROMPTCOT: Synthesizing Olympiad-level Problems for Mathematical Reasoning in Large Language Models
indexed_at: 2026-03-05T11:38:40.560340+00:00
---

## 摘要
PROMPTCOT 是一种合成奥林匹克级别数学问题的方法，通过（概念，rationale，问题）三元组训练问题生成模型。核心创新是引入 rationale 作为中间步骤使模型学习问题设计的内在逻辑，并通过双评估器拒绝采样筛选高质量数据。

## 关键概念
- Rationale: 解释如何从给定概念出发一步步构造出问题的思考过程
- 拒绝采样：使用两个独立 LLM 评估器对生成数据进行评估，仅保留一致认可的高质量样本
- 概念提取：使用 LLM 从种子问题中提取细化的数学概念以确保 rationale 贴近问题本质
- 三元组训练：以（概念，rationale，问题）数据结构训练问题生成模型

## 关联笔记
- 01KJBZQXDMK6QKQ7VMPCJQ4G4T.md: 介绍拒绝采样概念及其在 Post-training 飞轮中的应用，与 PROMPTCOT 的数据筛选机制相关
- 01KJBZWP1711J31GDNMNTAQW01.md: 研究思维链推理中的熵模式和高熵词元的作用，与 PROMPTCOT 的 rationale 推理生成相关
