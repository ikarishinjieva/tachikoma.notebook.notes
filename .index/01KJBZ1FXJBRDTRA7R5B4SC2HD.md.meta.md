---
note: 01KJBZ1FXJBRDTRA7R5B4SC2HD.md
title: 20230707 - chatglm微调, 使用SQL AST, 增强SQL规则验证(2)
indexed_at: 2026-03-05T08:51:40.331323+00:00
---

## 标签
ChatGLM-6B, 微调, SQL AST, 规则验证, PEFT, 训练结果

## 摘要
记录 2023 年 7 月 7 日 ChatGLM-6B 微调训练结果（198.66 epoch，train_loss 0.0171，耗时约 4 小时）。展示模型使用 SQL AST 进行 SQL 规则验证的正确案例与错误案例，错误集中在子查询 WhereClause 的识别。

## 关键概念
- ChatGLM-6B 微调: 使用 PEFT 框架对 ChatGLM-6B 模型进行监督微调
- SQL AST: SQL 抽象语法树，用于结构化分析 SQL 语句的各个组成部分
- PEFT Trainer: 参数高效微调训练器，负责模型训练和检查点保存
- 规则验证: 基于 AST 结构检查 SQL 是否符合预定义规则（如必须使用 WHERE 条件）

## 关联笔记
- 01KJBZ1FFFZ8ZMEZEAKXD9699E.md: 前一篇 (20230629) 首次使用 SQL AST 进行训练的实验记录
- 01KJBZ1S62NS39EME53Q8C4HY1.md: 后一篇 (20230712) 继续优化 SQL 规则审核能力的测试
- 01KJBZ23YC0VMQ76EFC97D2ZJX.md: 后续 (20230723) 从 p-tuning 换回 LoRA 的对比实验
