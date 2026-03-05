---
note: 01KJBZR8FB1T8BG3MPFYMDJ8KV.md
title: 20250205 - 阅读论文: AUTOREASON: AUTOMATIC FEW-SHOT REASONING DECOMPOSITION
indexed_at: 2026-03-05T11:22:53.122024+00:00
---

## 摘要
论文提出 AUTOREASON 方法，由强大模型生成思考路径（思考逻辑），引导较弱模型沿该路径进行推理。该方法主要目的是节省经济成本，通过推理分解实现模型能力迁移。

## 关键概念
- Few-Shot Reasoning Decomposition: 将复杂推理任务分解为可学习的步骤
- 思考路径: 由强大模型生成的推理逻辑序列
- 模型蒸馏: 用强模型指导弱模型学习的成本优化策略
- 自动推理分解: 自动生成推理步骤而非人工设计

## 关联笔记
- 01KJBZR8XAJASCJPA6NVR9TQBQ.md: DOTS 论文同样使用"思考路径"概念，由原子动作构成推理路径
- 01KJBZ63853E2DE7AXYHDQ1PGM.md: Chain-of-Thought Prompting 是通过 few-shot 教会 LLM 给出逻辑推断过程的基础方法
- 01KJBZRPASEJVBQPE28AAP50YZ.md: 涉及 reasoning path 和 DAG 构建，与推理路径生成相关
