---
note: 01KJBZRFKPHCGB3ETQ4MS9CVH0.md
title: 20250211 - 阅读论文*: CoRe: Solving Math Word Problems via Cooperative Reasoning induced Language Models
indexed_at: 2026-03-05T11:28:33.853800+00:00
---

## 标签
CoRe, 数学应用题, 合作推理, MCTS, 验证器, 困惑度阈值

## 摘要
CoRe 框架通过生成器与验证器（步骤验证器 Vstep、路径验证器 Vpath）的协作训练，使用 MCTS 搜索推理路径并结合困惑度阈值筛选高质量数据。通过迭代式自我思考阶段，将生成的优质推理路径合并到训练集中，持续提升模型在数学应用题上的零样本推理能力。

## 关键概念
- CoRe (合作推理): 生成器负责生成推理步骤，验证器评估质量并提供反馈，两者协作提升推理能力
- 验证器 (Verifier): 分为步骤验证器 Vstep 和路径验证器 Vpath，分别评估单步推理和完整推理路径的质量
- MCTS: 蒙特卡洛树搜索用于探索多种推理路径，通过选择 - 扩展 - 模拟 - 回溯四阶段生成候选答案
- 困惑度阈值 (PPL Threshold): 比较生成路径与人类标注路径的困惑度，过滤掉模型认为不自然的低质量推理
- 自我思考 (Self-Reflection): 迭代生成新训练数据，经分数和 PPL 双重过滤后合并到原训练集，循环优化模型

## 关联笔记
- 01KJBZRF4FT28F7NFSZD176TXQ.md: V-STaR 论文笔记，同天阅读，同样使用验证器训练方法但采用 DPO 而非 MCTS
- 01KJBZR2DKSYYSHN1HFGATWV4H.md: STaR 论文笔记，CoRe 借鉴了其迭代式自训练生成推理数据的核心思想
- 01KJBZR6R18RZDCX0A1GJ7HMB5.md: SRA-MCTS 论文笔记，同样使用 MCTS 优化推理过程但应用于代码生成而非数学问题
