---
note: 01KJBZ3BWM6KQH7FVEN8MHQN4F.md
title: 20230920 - 测试用codellama, 进行SQLe规则代码的微调
indexed_at: 2026-03-05T09:02:15.290864+00:00
---

## 摘要
记录使用 CodeLlama 对 SQLe 的 DDL 规则检查代码进行微调的实验过程，包含 4 条 DDL 规则的微调代码和 epoch=10、loss=0.1294 的训练结果。尝试通过增加类型定义上下文来修正 FIRST/AFTER 位置检查规则的生成效果。

## 关键概念
- SQLe: SQL 质量检查工具，提供 DDL/DML 规则校验能力
- AlterTableStmt: AST 节点类型，表示 ALTER TABLE 语句
- RuleHandlerInput: 规则处理函数输入结构，包含 Node、Res、Rule 等字段
- LoRA 微调：参数高效微调方法，用于适配代码生成任务

## 关联笔记
- 01KJBZ3CDY4S7H5QCATKFHAPYS.md: 20230923 的后续实验，跑通 SQLe 规则代码微调的单元测试
- 01KJBZ3AW9YYJVPKNE4SBSE7Y8.md: 20230913 的前置实验，首次尝试使用 CodeLlama 进行 SQL 规则代码生成
- 01KJBZN4KQV7NK5GRT46165QTR.md: 20241203 的模型微调知识整理，引用了本笔记作为案例
