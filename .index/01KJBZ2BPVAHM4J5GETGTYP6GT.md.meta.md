---
note: 01KJBZ2BPVAHM4J5GETGTYP6GT.md
title: 20230805 - chatglm 微调, 对sql进行规则审核 - 增加规则
indexed_at: 2026-03-05T08:56:56.225072+00:00
---

## 标签
ChatGLM 微调，SQL 规则审核，AST 解析，样本量优化，训练数据，embedding

## 摘要
记录使用 ChatGLM-6B 微调实现 SQL 规则到 AST 检查规则转换的探索过程。通过增加样本量（832→3328）和去除输入中的"..."符号，解决了规则匹配结果不稳定的问题。

## 关键概念
- ChatGLM-6B: 用于微调的大语言模型，实现 SQL 规则到 AST 检查规则的转换
- AST 检查规则：将 SQL 规则转换为抽象语法树的检查逻辑
- 样本量优化：通过增加训练数据（sql_rule_to_ast_rule_datas 从 832 增至 3328）提升模型准确性
- embedding: 向量表示方法，影响模型对特殊符号（如...）的处理稳定性

## 关联笔记
- 01KJBZZJ92R5GJ0VH1J4W3X93K.md: SQL 优化的 GRPO 训练数据生成过程，同属大模型训练 SQL 相关任务
- 01KJC00DJ026W6BGSW0TEW00GJ.md: SQL 优化的 GRPO 训练，涉及 SQL 规则处理与模型训练
- 01KJBZY39GJE38CB7BYSTRWMBX.md: SQL 优化的 GRPO 训练迭代，与 SQL 规则审核相关
