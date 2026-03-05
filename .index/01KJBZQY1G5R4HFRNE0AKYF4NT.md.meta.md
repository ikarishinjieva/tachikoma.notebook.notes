---
note: 01KJBZQY1G5R4HFRNE0AKYF4NT.md
title: 20250131 - 阅读论文: Query优化综述: A Survey of Query Optimization in Large Language Models
indexed_at: 2026-03-05T11:17:03.629108+00:00
---

## 摘要
该笔记综述了 LLM 中 Query 优化的四类核心方法：查询扩展、问题分解、查询消歧和查询抽象。每类方法列举了多种具体技术，如利用 LLM 生成知识扩展查询、迭代优化、rewrite-then-edit 框架消除歧义等。

## 关键概念
- Query Expansion: 利用 LLM 生成知识或预测内容来扩展原始查询，增强检索效果
- Question Decomposition: 将复杂查询分解为多个子问题，通过多方面检索器协同回答
- Query Disambiguation: 通过改写编辑查询或解决共指关系来减少歧义
- Query Abstraction: 将推理过程抽象为含变量的推理链，借助领域工具解决查询

## 关联笔记
- 01KJBZ5SKSN1ZNR6HMVAT6JG8H.md: Query2doc 论文详细讲解了查询扩展方法，与本笔记的 Query Expansion 直接相关
- 01KJBZR8XAJASCJPA6NVR9TQBQ.md: DOTS 论文定义了问题分解 (Query Decomposition) 和问题重写作为原子推理动作
- 01KJBZRB8197X3KFWB1F72P7VG.md: RICHES 论文提出检索与生成交织的方法，在 LLM 思考过程中动态生成检索词
