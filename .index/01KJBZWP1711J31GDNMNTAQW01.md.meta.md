---
note: 01KJBZWP1711J31GDNMNTAQW01.md
title: 20250807 - 阅读论文*: Beyond the 80/20 Rule: High-Entropy Minority Tokens Drive Effective Reinforcement Learning for LLM Rea
indexed_at: 2026-03-05T11:51:13.477253+00:00
---

## 摘要
论文发现 LLM 推理时 80% 词元为低熵（沿路执行），20% 为高熵（分叉路口决定推理方向）。实验证明适度提高高熵词元的生成随机性可提升推理性能，ms-swift 提供相应参数控制训练。

## 关键概念
- 熵模式: LLM 生成词元时的不确定性分布规律，分为低熵和高熵两类
- 低熵词元: 用于完成句子结构或单词拼写，起沿路执行作用
- 高熵词元: 逻辑连接词或转折词，在推理路径中起分叉路口作用
- 分叉路口: 高熵词元决定后续推理方向的关键节点

## 关联笔记
- 01KJBZWMER6XKW0WW7KF0Y4HXM.md: 同日期阅读的另一篇论文，讨论 RL 过程中熵坍塌现象及协方差正则化控制方法
