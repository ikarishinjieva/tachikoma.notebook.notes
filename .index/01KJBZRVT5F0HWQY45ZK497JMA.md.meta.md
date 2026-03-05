---
note: 01KJBZRVT5F0HWQY45ZK497JMA.md
title: 20250306 - 阅读论文*: SQL-o1: A Self-Reward Heuristic Dynamic Search Method for Text-to-SQL
indexed_at: 2026-03-05T11:32:51.125343+00:00
---

## 摘要
SQL-o1 是一种使用自奖励启发式动态搜索方法的 Text-to-SQL 解决方案，核心创新是在推理阶段应用 MCTS 替代传统逻辑拆解。通过修改版 UCT 策略平衡探索与利用，使用大模型置信度作为自奖励函数评估 SQL 质量，标准设置 500 次迭代实现复杂查询的高效生成。

## 关键概念
- MCTS 四阶段: 选择（UCT 策略）、扩展（Beam Search 生成候选）、模拟（生成完整 SQL 并评分）、反向传播（更新节点统计）
- 自奖励函数: 使用大模型对 SQL 的理解能力评估生成质量，无需外部资源
- 渐进式 SQL 生成策略: 训练阶段增强模型对 SQL 结构的理解
- 自适应迭代: 根据查询复杂度和树统计动态调整迭代次数（100-1000 次）

## 关联笔记
- 01KJBZR6R18RZDCX0A1GJ7HMB5.md: SRA-MCTS 论文笔记，同样在 MCTS 的扩展和评估阶段引入 LLM
- 01KJBZR3XNRAA8E4VR5EX8VSZM.md: rStar 方法笔记，使用 MCTS 和类人推理动作增强小模型推理
