---
note: 01KJBZJEG0K7W8SSNB4DD3SZQJ.md
title: 20241105 - 阅读论文*: A COMPARATIVE STUDY ON REASONING PATTERNS OF OPENAI’S O1 MODEL
indexed_at: 2026-03-05T10:54:38.856333+00:00
---

## 标签
o1 模型，推理模式，论文分析，大语言模型，推理机制，分治策略

## 摘要
本笔记总结了一篇关于 OpenAI o1 模型推理模式的 comparative study 论文，分析了 o1 在数学、代码等任务上优于基线方法的原因。论文通过人工分析归纳出六种推理模式（系统分析、方法复用、分治、自我修正、上下文识别、强调约束），其中分治和自我修正是最主要的推理模式。

## 关键概念
- 分治 (DC): 将复杂问题分解成多个子问题，通过解决子问题构建最终答案
- 自我修正 (SR): 在推理过程中不断评估和修正自己的推理步骤
- 方法复用 (MR): 复用已知的经典方法解决问题
- 系统分析 (SA): 先分析问题整体结构，再确定算法和数据结构
- 强调约束 (EC): 强调题目中的约束条件以规范输出

## 关联笔记
- 01KJBZRPASEJVBQPE28AAP50YZ.md: 同样分析 o1 模型的推理机制，探讨了 reasoning tokens 与决策树遍历方式的关系
- 01KJBZRVT5F0HWQY45ZK497JMA.md: SQL-o1 论文使用 MCTS 进行搜索推理，与 o1 的推理模式研究形成对比
- 01KJBZY0MAYHH592A9NG68XWV3.md: 研究小模型 RL 推理训练，以 o1 系列作为顶级模型的成功范式参考
