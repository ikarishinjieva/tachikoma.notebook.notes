---
note: 01KJBZRPASEJVBQPE28AAP50YZ.md
title: 20250227 - 阅读论文*: 迈向复现 OpenAI o1 的一小步：Steiner 开源模型解析
indexed_at: 2026-03-05T11:30:33.993713+00:00
---

## 摘要
解析 Steiner 开源模型复现 OpenAI o1 的核心方法，包括通过 reasoning tokens 线性增长推断 o1 采用线性遍历而非树形并发搜索。介绍两种带 backtracking 的 CoT 数据合成方法及基于 DAG 节点权重的步级 reward 设计。

## 关键概念
- 线性决策树遍历: o1 使用线性而非并发方式遍历决策树，可复用现有推理基础设施
- Backtracking CoT: 通过随机截断和隐藏答案让 LLM 生成带回溯的推理链数据
- DAG 推理图: 将推理 steps 聚类建图为有向无环图，支持多项式量级 path 采样
- 步级 Reward: 基于 DAG 节点的入/出边数和距离问题/答案的距离加权计算 reward

## 关联笔记
- 01KJBZY0MAYHH592A9NG68XWV3.md: 同样研究小模型上通过 RL 训练推理能力，涉及 o1 范式和 GRPO 算法
- 01KJBZRVT5F0HWQY45ZK497JMA.md: SQL-o1 使用 MCTS 进行动态搜索，与决策树搜索方法相关
- 01KJBZYAS3THKM4CYDK96M7AQV.md: GLM-4.5 的迭代式自蒸馏流程与 Steiner 的 SFT->RL->生成轨迹方法相似
