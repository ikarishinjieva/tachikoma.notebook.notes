---
note: 01KJBZ9S1E9ZR6TKD7MPA225EN.md
title: 20240612 - 阅读论文: SYNCHROMESH: RELIABLE CODE GENERATION FROM PRE-TRAINED LANGUAGE MODELS
indexed_at: 2026-03-05T10:03:47.951532+00:00
---

## 摘要
Synchromesh 论文提出通过目标相似性调整 (TST) 和受约束语义解码 (CSD) 两个组件优化 LLM 生成代码的可靠性。CSD 使用 Brzozowski 导数动态计算合法后续 token 集合，确保生成代码符合语法和语义约束。

## 关键概念
- 目标相似性调整 (TST): 通过结构相似性而非词句相似性找到相似代码片段
- 受约束语义解码 (CSD): 通过定义约束消除代码实现错误，保证生成代码遵循规范
- Brzozowski 导数: 在给定代码前缀和约束下，后续可用 token 集合的形式化表示
- 完成引擎 (CE): 用于在代码生成过程中定义和实施约束的灵活机制

## 关联笔记
- 01KJBZR6R18RZDCX0A1GJ7HMB5.md: 同为代码生成可靠性优化的论文阅读笔记，使用 MCTS 优化思考过程
- 01KJBZD3K42PBXXQ4KSTFQDP4N.md: CODEAGENT 论文笔记，同样关注增强代码生成的可靠性与工具集成
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: 包含多种代码生成方法的总结，涉及 RAG、概率排序等可靠性提升技术
