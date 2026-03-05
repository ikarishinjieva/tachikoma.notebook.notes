---
note: 01KJBZ3AKB9P49TFR3CP9PB0EQ.md
title: 20230901 - chatglm 微调, 对sql进行规则审核 - 使用randgen生成训练SQL
indexed_at: 2026-03-05T09:01:02.075637+00:00
---

## 摘要
记录使用 randgen 生成训练 SQL 数据对 ChatGLM 进行微调，针对"恒等式"规则（WHERE 子句不能包含恒等条件）进行训练。数字匹配效果较好，但文字判断错误率高，部分用例输出不稳定，通过增加反例对比数据后大部分问题得到改善。

## 关键概念
- randgen: 用于生成大量训练 SQL 的工具，可定向生成特定规则的测试用例
- 恒等式规则: 检查 Select 语句的 WhereClause 中是否包含恒等条件（如 1=1, 'abc'='abc'）
- epoch 调优: 需要将 loss 降到 0.001+，当前 epoch=6 使用 4 轮训练
- 反例对比: 通过生成对立样例（必须使用/不能使用恒等条件）增强模型辨别能力

## 关联笔记
- 01KJBZN4KQV7NK5GRT46165QTR.md: 模型微调知识整理，汇总了 SQL 审核微调的完整尝试历程
- 01KJBZ1S62NS39EME53Q8C4HY1.md: 20230712 的 SQL 规则审核微调笔记，系列早期版本记录 AST 规则判断问题
- 01KJBZ391BKTMFGSSBDRWEWS5H.md: 调整配置文件生成恒等条件的相关记录
