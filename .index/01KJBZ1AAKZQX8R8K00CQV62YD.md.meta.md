---
note: 01KJBZ1AAKZQX8R8K00CQV62YD.md
title: 20230616 - 对chatglm进行微调, 用于替换SQL审核引擎
indexed_at: 2026-03-05T08:48:49.384692+00:00
---

## 摘要
记录使用 ChatGLM-Efficient-Tuning 对 ChatGLM-6B 进行微调以替换 SQL 审核引擎的探索过程。发现模型存在边界值不敏感、Prompt 格式敏感、无法理解局部规则等问题，并通过调整训练参数（epoch、batch size）部分解决。

## 关键概念
- ChatGLM-Efficient-Tuning: 智谱 AI 提供的 ChatGLM 高效微调工具库
- Prompt 格式敏感性: 输入中"SQL"字样、分号等细节会显著影响模型输出稳定性
- 边界值不敏感: 模型对 limit=1000 与 limit=1001 等临界值判断不稳定
- 局部规则理解: 模型难以同时处理复合规则中的多个条件
- 训练参数调优: 增加 epoch 并缩小 batch size 至 1 可改善某些不稳定现象

## 关联笔记
- 01KJBZN4KQV7NK5GRT46165QTR.md: 模型微调知识汇总，索引了本笔记及后续所有 ChatGLM 微调探索记录
- 01KJBZ1C96JJPH6H5RRH2V7T8S.md: 深入研究微调样本对模型结果冲突的影响
- 01KJBZ1FFFZ8ZMEZEAKXD9699E.md: 后续尝试引入 SQL AST 增强规则验证能力的探索
