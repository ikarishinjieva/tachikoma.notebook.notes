---
note: 01KJBZB0PHX4M0QCQ6CKGC7NJQ.md
title: 20240723 - 知识图谱的关系抽取
indexed_at: 2026-03-05T10:24:28.923617+00:00
---

## 标签
知识图谱，关系抽取，ConceptNet，DBpedia，Schema.org, MySQL 文档

## 摘要
探索将 MySQL 文档转换为知识图谱以增强 RAG 召回效果，测试了 ConceptNet、DBpedia、Schema.org 三种知识表示方案的关系词/谓词设计。结论显示朴素 LLM 三元组抽取不实用，需结合领域特定的关系建模。

## 关键概念
- ConceptNet: 基于常识的知识图谱，使用 34 个固定关系词进行语义关联
- DBpedia: 基于 RDF 的知识库，通过 SPARQL 查询，包含 56354 个谓词
- Schema.org: 结构化数据标记标准，用于定义实体类型和属性
- 关系抽取：从非结构化文本中提取实体间语义关系并构建三元组

## 关联笔记
- 01KJBZB5Z3B527PJ4RY1B179WE.md: 同系列的后续实验，继续使用 LLM 尝试构建知识图谱
- 01KJBZBMFN7QZYG0SXH1XM3S7Z.md: LlamaIndex 知识图谱增强 RAG 的实现方案分析
- 01KJBZANGZVY92HGNK5652A6G8.md: 知识图谱补全算法 PCST 的研究笔记
