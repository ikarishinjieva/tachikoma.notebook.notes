---
note: 01KJBZR3XNRAA8E4VR5EX8VSZM.md
title: 20250202 - 阅读论文*: rStar: MUTUAL REASONING MAKES SMALLER LLMS STRONGER PROBLEM-SOLVERS
indexed_at: 2026-03-05T11:20:22.444808+00:00
---

## 摘要
rStar 通过五种类人推理动作增强 MCTS 搜索，使小语言模型能够探索多样化推理路径。引入第二个小模型作为判别器进行相互一致性验证，通过交叉验证提高推理轨迹选择的准确性。

## 关键概念
- 蒙特卡洛树搜索 (MCTS): 通过选择、扩展、模拟、反向传播四步骤探索推理路径
- 类人推理动作集: 五种动作（提议思路、补全步骤、子问题、重答、重述问题）模拟人类推理
- 相互一致性验证: 用 SLM2 补全轨迹并与 SLM1 答案比对，验证推理有效性
- UCT 算法: 平衡探索与利用的节点选择策略
- 自一致性: 通过多数投票确定终止节点置信度

## 关联笔记
- 01KJBZR6R18RZDCX0A1GJ7HMB5.md: SRA-MCTS 同样使用 MCTS 优化文本推理过程，但应用于代码生成
- 01KJBZRFKPHCGB3ETQ4MS9CVH0.md: CoRe 使用 MCTS 生成解题思路，并用验证器评估路径质量
