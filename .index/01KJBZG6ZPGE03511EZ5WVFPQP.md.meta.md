---
note: 01KJBZG6ZPGE03511EZ5WVFPQP.md
title: 20241007 - 阅读论文*: Tool-Assisted Agent on SQL Inspection and Refinement in Real-World Scenarios
indexed_at: 2026-03-05T10:47:02.652478+00:00
---

## 标签
Text-to-SQL, Agent, 纠错工具, 数据库检索器, 错误检测器, 函数调用

## 摘要
论文提出在 Text-to-SQL 过程中引入两种辅助纠错工具：检索器（Database Retriever）用于发现条件子句中常量值与实际值不匹配，检测器（Error Detector）用于检测 SQL 语法错误和逻辑约束违反。使用 LLM 函数调用生成 SQL 能增强结构化正确性，并通过 QA() 函数处理嵌套子查询。

## 关键概念
- 数据库检索器 (Database Retriever): 使用 SimCSE 模型计算条件子句中值与数据库单元格的语义相似度，返回最相似的单元格作为反馈
- 错误检测器 (Error Detector): 检查函数调用序列是否符合 SQL 语法规则和数据库 schema，检测外键关系、JOIN 操作、列类型匹配等错误
- 函数调用序列: 将 SQL 分解为 add_answer、add_from、add_where 等函数调用，简化 LLM 生成结构正确 SQL 的任务
- QA() 函数: 专门处理子查询的函数，使 LLM 能逐步生成包含嵌套子查询的复杂 SQL
- Spider-Mismatch 数据集: 从 Spider 系列数据集衍生，专门突出用户问题中的值与数据库实际值之间的差异

## 关联笔记
- 01KJBZD3K42PBXXQ4KSTFQDP4N.md: 同为 Tool-Integrated Agent 系统的论文阅读笔记，CODEAGENT 也使用工具增强代码生成
