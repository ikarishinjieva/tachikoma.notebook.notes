---
note: 01KJBZBNS3200SZ8EH9C94S1ZB.md
title: 20240726 - 阅读论文: Knowledge Graph Enhanced Retrieval-Augmented Generation for Failure Mode and Effects Analysis
indexed_at: 2026-03-05T10:27:11.794688+00:00
---

## 标签
知识图谱，RAG，检索增强生成，FMEA，信息块提取，混合检索

## 摘要
笔记分析了一篇将知识图谱与 RAG 结合的论文，提出根据行业规则从图谱节点转换信息块的混合检索方案。该方法针对 FMEA 场景使用 DFS 算法以失效影响节点为根提取信息块，结合图谱直接搜索与向量相似性检索实现关系模糊查询。

## 关键概念
- 知识图谱增强检索：利用图谱结构关系辅助传统向量检索，提升多跳推理能力
- 信息块：从知识图谱节点按规则提取并整合的文本块，用于向量检索
- DFS 算法：深度优先搜索，用于遍历图谱中相连节点并提取关联信息
- 混合检索：结合图谱结构化搜索与向量相似性检索的互补方案
- FMEA：失效模式与影响分析，具有规整表格形式的行业数据场景

## 关联笔记
- 01KJBZF7F960C9BRB1RD259XZM.md: 探讨 Graph 与 RAG 结合的适用场景和问题范围
- 01KJBZB0PHX4M0QCQ6CKGC7NJQ.md: 记录将文档转换为知识图谱辅助 RAG 召回的探索
- 01KJBZJCR5P60FXY3BH47TAFH4.md: HippoRAG 同样使用知识图谱增强检索，采用 PageRank 进行图搜索
