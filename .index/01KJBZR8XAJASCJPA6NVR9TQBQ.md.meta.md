---
note: 01KJBZR8XAJASCJPA6NVR9TQBQ.md
title: 20250206 - 阅读论文: DOTS: LEARNING TO REASON DYNAMICALLY IN LLMS VIA OPTIMAL REASONING TRAJECTORIES SEARCH
indexed_at: 2026-03-05T11:23:35.597788+00:00
---

## 摘要
DOTS 通过定义原子推理动作模块（分析层、解答层、验证层），使用搜索算法为每个问题找到最优推理轨迹。通过微调规划器 LLM（外部或内部化）实现根据问题特征动态选择最优推理路径，提升 LLM 推理性能。

## 关键概念
- 原子推理动作：构成推理路径的基本模块，分为分析层（问题重写/分解）、解答层（CoT/PoT）、验证层（自我验证）
- 最优推理轨迹：通过迭代评估筛选出的成功率最高且路径最短的推理动作序列
- 规划器 LLM：微调后能根据输入问题预测最优推理轨迹的模型，支持外部独立部署或内部化集成
- 轨迹解释：用更强 LLM（如 GPT-4o）为最优轨迹生成自然语言解释，说明选择理由以提高可解释性

## 关联笔记
- 01KJBZR6R18RZDCX0A1GJ7HMB5.md: SRA-MCTS 同样使用 MCTS 搜索最优推理路径，但专注于代码生成场景
- 01KJBZR3XNRAA8E4VR5EX8VSZM.md: rStar 使用 MCTS 和五种类人推理动作探索推理轨迹，并用判别器验证路径正确性
- 01KJBZRFKPHCGB3ETQ4MS9CVH0.md: CoRe 使用 MCTS 生成推理思路，并用步骤/路径验证器评估路径质量
