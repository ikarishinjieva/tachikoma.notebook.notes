---
note: 01KJBZ9G97AWEPERJRVQZTYGW3.md
title: 20240521 - 使用RAG的方法进行SQLe代码生成
indexed_at: 2026-03-05T09:58:34.880990+00:00
---

## 摘要
探索使用 RAG 方法进行 SQL 规则检查代码生成，针对复杂规则（SQLE00039/55）需要长链推导的问题。模仿 REACT 论文思路，将复杂规则拆解为多个 FIND/GENERATE 步骤，逐步检索样例代码并整合生成最终代码。

## 关键概念
- REACT: 推理与行动协同的语言模型框架，通过思考 - 行动循环解决复杂任务
- AceCoder: 利用现有代码增强代码生成的方法，通过检索相似代码作为示例
- FIND 行动: 在特定类型数据库中查找满足某功能需求的样例代码
- GENERATE 行动: 整合多个检索到的样例代码生成目标代码
- 规则拆解: 将复合规则描述分解为独立子任务逐一处理

## 关联笔记
- 01KJBZ9B7WN6ADZSF3J8ZN31X9.md: REACT 论文阅读笔记，为本笔记提供方法论基础
- 01KJBZ9C1ZG41EB9NF0W87ZXSP.md: AceCoder 论文阅读笔记，提供利用现有代码增强生成的思路
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: 同一系列笔记，同样引用了 REACT 和 AceCoder 论文进行 SQL 代码生成探索
