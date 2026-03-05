---
note: 01KJBZ23YC0VMQ76EFC97D2ZJX.md
title: 20230723 - chatglm 微调, 对sql进行规则审核 [3]
indexed_at: 2026-03-05T08:55:12.318434+00:00
---

## 摘要
记录从 P-Tuning 切换回 LoRA 进行 ChatGLM-6B 微调的实验过程。主要问题是模型对含子查询 +ORDER BY+LIMIT 的 SQL 生成 AST 时出现解析错误，导致规则判断失效。尝试通过增加 order+limit+ 子查询的 SQL 训练样本来改进，但效果有限。

## 关键概念
- LoRA: 低秩适配器微调方法，相比 P-Tuning 训练更稳定
- AST(抽象语法树): 将 SQL 解析为结构化表示，用于规则验证
- P-Tuning: 提示微调方法，本笔记中效果不如 LoRA
- 子查询嵌套: SQL 规则审核中的难点场景，易导致 AST 解析错误

## 关联笔记
- 01KJBZN4KQV7NK5GRT46165QTR.md: 模型微调知识整理，包含本系列完整索引
- 01KJBZ1Z4PSVWAT3RBRKWB688P.md: 系列 [2]，使用 P-Tuning 进行 SQL 规则审核的对比实验
- 01KJBZ1FFFZ8ZMEZEAKXD9699E.md: 系列 [1]，首次尝试使用 SQL AST 增强规则验证
