---
note: 01KJBZ24FPFXBE9808JY2XJ5RS.md
title: 20230729 - chatglm 微调, 对sql进行规则审核 - 尝试去除分步骤
indexed_at: 2026-02-27T10:33:38.168834+00:00
---

Let me analyze this note about ChatGLM fine-tuning for SQL rule auditing. I need to extract:

1. Tags (3-6 keywords)
2. Summary (2-3 sentences)
3. Key concepts (3-5 concepts with explanations)
4. Related notes (0-3 from the provided list)

The note is about:
- Fine-tuning ChatGLM-6B for SQL rule auditing
- Trying to remove step-by-step approach and only train on final results
- Issues with model making incorrect judgments
- Adding match training data improves accuracy
- Hypothesis about why match data helps (weight distribution)

Looking at the related notes list, I need to find any that might relate to:
- Machine learning/AI training
- SQL analysis
- Clickhouse (some notes mention this)
- Database performance/analysis

From the list, the most potentially related notes seem to be:
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: Clickhouse集群 Distributed Engine 学习
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 20210415 - Clickhouse MaterializedMySQL 学习
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 20210531 - Clickhouse Join 研究
- 01KJBYDD58AJTW5NKT3GET7B89.md: 20221022 - 旧：Clickhouse+学习笔记
- 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: 20210622 - Clickhouse distributed join 学习
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 20210429 - SQLLancer 学习


- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 20210429 - SQLLancer 学习 (SQL analysis tool - most related)

The SQLLancer note appears most relevant since it deals with SQL analysis, which aligns with the SQL rule auditing focus of this note.## 标签
ChatGLM, 微调, SQL 规则审核, AST 分析, 训练策略, 大模型

## 摘要
记录尝试对 ChatGLM-6B 进行微调以审核 SQL 是否符合规则，探索去除分步骤只训练总结论的方法。发现单独添加 match 训练数据能提高判断准确性，推测是因为输出权重分配问题。

## 关键概念
- AST 检查规则: 将 SQL 规则转换为抽象语法树的检查逻辑
- 分步骤训练: 让模型输出完整推理过程而非仅最终结论
- 输出权重: 模型训练中不同输出内容对损失函数的影响程度
- Match 数据: 单独的训练样本，输入长输出短（是/否判断）

## 关联笔记
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 同为 SQL 分析工具学习相关
