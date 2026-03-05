---
note: 01KJBZ1S62NS39EME53Q8C4HY1.md
title: 20230712 - chatglm 微调, 对sql进行规则审核
indexed_at: 2026-03-05T08:52:24.816899+00:00
---

## 摘要
记录 ChatGLM-6B 微调过程中 SQL 规则判断的两个典型问题：AST 规则判断错误（将含 OrderByClause 的 AST 误判为"否"）和训练数据干扰（"否"错误匹配到 limit 规则的问答模式）。通过增加 AST 操作判断示例数据和更多针对性训练样本来修正模型行为。

## 关键概念
- AST 规则判断: 模型需正确识别 Select 语句是否直接使用 OrderByClause
- 训练数据干扰: 模型将"否"结论错误匹配到相似规则（如 limit 规则）的问答模式
- 数据增强: 增加 AST 操作判断的正负样本对，提升规则判断准确性

## 关联笔记
- 01KJBZN4KQV7NK5GRT46165QTR.md: 整个 ChatGLM 微调系列的总索引，包含此笔记在内的所有实验记录
- 01KJBZ1FFFZ8ZMEZEAKXD9699E.md: 前序实验，首次尝试使用 SQL AST 增强 SQL 规则验证
- 01KJBZ1Z4PSVWAT3RBRKWB688P.md: 后续实验 [2]，继续解决 order by 规则判断问题并增加训练数据
