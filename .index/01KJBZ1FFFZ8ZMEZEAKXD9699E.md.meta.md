---
note: 01KJBZ1FFFZ8ZMEZEAKXD9699E.md
title: 20230629 - chatglm微调, 使用SQL AST, 增强SQL规则验证
indexed_at: 2026-02-27T10:31:37.383836+00:00
---

用户希望我分析这篇关于 ChatGLM 微调和 SQL AST 的笔记，生成索引元数据。让我仔细阅读笔记内容，提取关键信息。

笔记核心内容：
1. 使用 SQL AST（抽象语法树）进行训练
2. 从 SQLite 抽取 SQL，用 sqltree 生成 SQL AST
3. 训练 40 epoch 后效果不错
4. 下一步是用 SQL AST 增强 SQL 规则验证
5. 遇到的问题：训练数据权重分配问题
6. 展示了 AST 如何帮助分析 SQL 的示例对比

关键概念：
- SQL AST（SQL 抽象语法树）
- ChatGLM 微调
- SQL 规则验证
- sqltree 项目
- 训练数据权重

关联笔记分析：
从提供的笔记列表来看，这些笔记主要是 2021 年的 MySQL、Clickhouse、数据库相关的笔记。当前笔记是 2023 年 6 月关于 ChatGLM 微调和 SQL AST 的内容。

可能的关联：
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 20210429 - SQLLancer 学习（都与 SQL 分析相关）
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 20210606 - 从 binlog 中分离 SQL, 进行统计（都涉及 SQL 处理）
- 01KJBYDD58AJTW5NKT3GET7B89.md: 20221022 - 旧：Clickhouse+ 学习笔记（时间上最接近，但内容关联度一般）


- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: SQLLancer 学习（SQL 分析工具，与 SQL AST 分析最相关）

这篇笔记聚焦于通过 SQL AST 增强模型对 SQL 结构的理解和规则验证能力，与早期的 SQL 分析工具学习有一定的技术延续性。## 标签
ChatGLM 微调，SQL AST，SQL 规则验证，大模型训练，sqltree

## 摘要
记录使用 SQL AST 对 ChatGLM-6B 进行微调的实践过程，训练 40 epoch 后模型具备生成和分析 SQL AST 的能力。探讨了如何在此基础上增强大模型的 SQL 规则判断能力，以及训练数据权重分配的问题。

## 关键概念
- SQL AST: SQL 抽象语法树，用于结构化表示 SQL 语句的语法结构
- sqltree: 用于将 SQL 语句解析生成 AST 的开源项目
- 训练数据权重: 不同能力训练数据在微调中的占比影响最终效果

## 关联笔记
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: SQLLancer 学习，同属 SQL 分析工具相关研究
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 从 binlog 中分离 SQL 进行统计，涉及 SQL 抽取处理
