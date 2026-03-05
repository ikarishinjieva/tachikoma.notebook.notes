---
note: 01KJBZ1FFFZ8ZMEZEAKXD9699E.md
title: 20230629 - chatglm微调, 使用SQL AST, 增强SQL规则验证
indexed_at: 2026-03-05T08:51:13.588392+00:00
---

## 摘要
记录使用 sqltree 将 SQL 转换为 AST 结构来训练 ChatGLM-6B 模型的实验，40 epoch 训练后模型已具备生成 SQL AST 的能力。探讨如何在已有 SQL AST 能力基础上增强模型的 SQL 规则判断能力，面临训练数据权重分配和训练效率的权衡问题。

## 关键概念
- SQL AST: SQL 的抽象语法树表示，将 SQL 语句解析为结构化的树形数据结构
- 训练数据权重: 不同能力训练数据在模型学习过程中的影响力比例
- 规则验证增强: 在模型已掌握 SQL AST 生成能力后，叠加 SQL 规则判断能力的训练策略

## 关联笔记
- 01KJBZ1FXJBRDTRA7R5B4SC2HD.md: 后续实验记录，继续进行 SQL AST 增强 SQL 规则验证的训练迭代
- 01KJBZN4KQV7NK5GRT46165QTR.md: 模型微调知识整理笔记，引用本笔记作为 SQL 领域微调的研究案例
