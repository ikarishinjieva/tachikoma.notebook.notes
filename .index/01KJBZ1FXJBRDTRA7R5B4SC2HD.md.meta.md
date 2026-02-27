---
note: 01KJBZ1FXJBRDTRA7R5B4SC2HD.md
title: 20230707 - chatglm微调, 使用SQL AST, 增强SQL规则验证(2)
indexed_at: 2026-02-27T10:31:46.414521+00:00
---

用户要求我分析一篇关于 ChatGLM 微调的笔记，生成索引元数据。让我仔细分析笔记内容：

1. 笔记标题：20230707 - chatglm 微调，使用 SQL AST, 增强 SQL 规则验证 (2)
2. 内容主要是：
   - 训练结果（2023 年 7 月 7 日进行的训练）
   - 训练指标（loss、epoch、runtime 等）
   - 一个错误的示例，展示 ChatGLM-6B 对 SQL 规则验证的分析过程
   - 使用 AST（抽象语法树）来分析 SQL 是否符合规则

核心主题：
- ChatGLM-6B 模型微调
- SQL 规则验证
- AST（抽象语法树）分析
- 训练结果和错误分析

关联笔记分析：
从笔记列表看，大部分是 2021 年的 MySQL、Clickhouse、数据库相关的笔记。这篇笔记是 2023 年的，关于 ChatGLM 微调和 SQL 规则验证。

可能相关的：
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 20210429 - SQLLancer 学习（SQL 相关）
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 20210606 - 从 binlog 中分离 SQL，进行统计（SQL 相关）

但这些都是 2021 年的数据库相关笔记，与 2023 年的 ChatGLM 微调笔记关联度不高。

现在生成索引元数据。## 标签
ChatGLM-6B, SQL 规则验证, AST 抽象语法树, 模型微调, 训练结果

## 摘要
记录 2023 年 7 月 7 日 ChatGLM-6B 模型微调的训练结果，epoch 达 198.66，train_loss 为 0.0171。展示了模型使用 AST 分析 SQL 是否符合规则的错误示例，主查询有 WHERE 但子查询缺少 WHERE 条件时模型判断有误。

## 关键概念
- AST: 抽象语法树，用于结构化表示 SQL 语句的语法结构
- SQL 规则验证: 通过检查 AST 判断 SQL 是否符合预定义规则（如 select 必须使用 where 条件）
- PEFT: 参数高效微调方法，用于训练 ChatGLM-6B 模型
- Subselect: 子查询，嵌套在父查询中的 SELECT 语句

## 关联笔记
无
