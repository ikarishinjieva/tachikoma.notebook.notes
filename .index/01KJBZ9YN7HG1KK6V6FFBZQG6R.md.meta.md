---
note: 01KJBZ9YN7HG1KK6V6FFBZQG6R.md
title: 20240613 - 阅读论文: When Not to Trust Language Models: Investigating Effectiveness of Parametric and Non-Parametric Memorie
indexed_at: 2026-03-05T10:07:07.345855+00:00
---

## 摘要
论文分析 LLM 对事实性知识的记忆能力，发现模型性能与主题实体流行度正相关，且对低流行度知识的记忆存在明显局限。提出自适应检索方法：根据实体流行度阈值动态决定是否启用外部检索，在答案准确性与检索成本之间取得平衡。

## 关键概念
- 参数化记忆: LLM 通过预训练编码到自身参数中的事实性知识
- 主题实体流行度: 实体在训练数据中出现的频率，可预测 LLM 的记忆效果
- 自适应检索: 根据实体流行度阈值动态切换使用内部记忆或外部检索的策略
- 长尾现象: LLM 对高流行度知识记忆良好，对低流行度知识记忆有限的现象

## 关联笔记
- 01KJBZA2DWNTFGGEGZBWV6V86R.md: 同一研究方向的后续论文，提出用知识边界模型判断是否需要检索
- 01KJBZWJBAGGD1TA8SGM6YZXR4.md: KAG-Thinker 框架也采用知识边界判定策略来决定是否启动外部检索
