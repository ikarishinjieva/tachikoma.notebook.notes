---
note: 01KJBZ1Z4PSVWAT3RBRKWB688P.md
title: 20230717 - chatglm 微调, 对sql进行规则审核 [2]
indexed_at: 2026-02-27T10:32:27.602182+00:00
---

用户要求我分析一篇关于 ChatGLM 微调和 SQL 规则审核的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20230717 - chatglm 微调，对 sql 进行规则审核 [2]
2. 内容主要是关于使用 ChatGLM-6B 模型来判断 SQL 是否符合定义的规则，涉及 AST 分析

关键信息：
- 背景：重构数据生成器，从最终结果分离出每个步骤的训练数据
- 问题：使用 ChatGLM-6B 判断 SQL 是否符合规则
- 涉及技术：ChatGLM-6B、SQL 规则审核、AST（抽象语法树）分析

标签应该包括：ChatGLM、SQL 审核、规则引擎、AST 分析、微调

摘要：描述使用 ChatGLM-6B 模型进行 SQL 规则审核的微调工作，通过 AST 分析判断 SQL 是否符合定义的规则。

关键概念：
- ChatGLM-6B: 开源对话大语言模型
- AST: 抽象语法树，用于 SQL 语法分析
- SQL 规则审核: 验证 SQL 是否符合预定义规则

关联笔记：查看其他笔记列表，主要是 MySQL、ClickHouse、数据库相关的笔记。这篇笔记关于 SQL 规则审核和 ChatGLM 微调，与数据库相关的笔记可能有关联，但直接关联的不多。

看了一些笔记标题，没有直接关于 SQL 审核或 ChatGLM 的内容，所以关联笔记可能是「无」或者找一些 SQL 相关的。

让我找找可能相关的：
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: Clickhouse 集群 Distributed Engine 学习 - SQL 相关
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: Clickhouse MaterializedMySQL 学习 - SQL 相关
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 从 binlog 中分离 SQL, 进行统计 - SQL 处理相关

最后一篇关于从 binlog 分离 SQL 进行统计的笔记与当前笔记的 SQL 处理主题最为贴近。## 标签
ChatGLM, SQL 审核, AST 分析, 大模型微调, 规则引擎

## 摘要
记录使用 ChatGLM-6B 模型进行 SQL 规则审核的微调工作，通过重构数据生成器从最终结果中分离训练数据。核心方法是让模型解析 SQL 生成 AST，并基于 AST 判断 SQL 是否符合预定义规则。

## 关键概念
- ChatGLM-6B: 开源对话大语言模型，用于 SQL 规则判断任务
- AST（抽象语法树）: SQL 语法结构化的树形表示，用于规则分析
- SQL 规则审核: 验证 SQL 语句是否符合预定义的语法或安全规则

## 关联笔记
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 都涉及从原始数据中提取和分析 SQL 语句
