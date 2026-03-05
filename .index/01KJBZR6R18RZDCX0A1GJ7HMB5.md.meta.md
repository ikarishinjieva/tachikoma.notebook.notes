---
note: 01KJBZR6R18RZDCX0A1GJ7HMB5.md
title: 20250204 - 阅读论文*: SRA-MCTS: Self-driven Reasoning Augmentation with Monte Carlo Tree Search for Code Generation
indexed_at: 2026-03-05T11:22:21.720141+00:00
---

## 摘要
SRA-MCTS 将 LLM 引入蒙特卡洛树搜索的扩展和评估步骤，通过模拟 rollout 和多维度评估机制优化代码生成中的文本思考过程。相比标准 MCTS，该算法在模拟策略、评估粒度、动态终止三个方面进行改进，更适用于复杂推理任务。

## 关键概念
- SRA-MCTS: 结合模拟 rollout 和 MCTS 的算法，通过 LLM 生成和评估推理路径
- 探索与利用平衡: 选择策略优先选择平均得分高且访问次数少的节点
- 模拟 Rollout: 从扩展节点开始用 LLM 生成完整推理路径直到终止状态
- 多维度评估: 从单步正确性、整体连贯性、完整性、整体正确性四方面评分
- 动态终止: LLM 根据推理路径完整性动态决定是否结束模拟

## 关联笔记
- 01KJBZR8XAJASCJPA6NVR9TQBQ.md: DOTS 论文同样研究最优推理轨迹搜索，通过原子动作组合构建推理路径
- 01KJBZRPASEJVBQPE28AAP50YZ.md: Steiner 模型解析涉及推理树遍历和 backtracking 策略，与 MCTS 搜索机制相关
