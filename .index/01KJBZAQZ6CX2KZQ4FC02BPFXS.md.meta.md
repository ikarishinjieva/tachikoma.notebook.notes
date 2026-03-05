---
note: 01KJBZAQZ6CX2KZQ4FC02BPFXS.md
title: 20240718 - ChatDBA: 如何使用知识类的文档
indexed_at: 2026-03-05T10:22:20.226681+00:00
---

## 摘要
记录 ChatDBA 项目中使用知识文档的两种方法对比：朴素向量检索（bge-m3 + haystack）与图方法（Neo4j + MySQL 知识图谱）。测试发现图查询无法命中问题，核心问题是知识图谱构建规范与查询规范不一致，需参考 ConceptNet 等 NLP 库设计关系词体系。

## 关键概念
- 知识图谱构建规范: 实体和关系的建模方式需与查询模式匹配才能命中节点
- ConceptNet 关系词: 34 个标准关系词（如 UsedFor、HasProperty）用于语义关联建模
- 向量检索: 使用 bge-m3 向量化文档，在 haystack In memory 存储中匹配
- Neo4j 图数据库: 存储 MySQL 8.4 手册生成的图谱，支持 Cypher 查询

## 关联笔记
- 01KJBZB0PHX4M0QCQ6CKGC7NJQ.md: 后续实验，使用 LLM 进行知识图谱关系抽取的尝试
- 01KJBZB5Z3B527PJ4RY1B179WE.md: 知识图谱关系抽取的第二轮实验与优化
- 01KJBZJCR5P60FXY3BH47TAFH4.md: 研究 HippoRAG 如何利用知识图谱进行段落检索
