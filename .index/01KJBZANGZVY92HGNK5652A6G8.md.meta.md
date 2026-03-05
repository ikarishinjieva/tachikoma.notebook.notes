---
note: 01KJBZANGZVY92HGNK5652A6G8.md
title: 20240716 - 阅读论文: 关于知识图谱算法PCST (非论文)
indexed_at: 2026-03-05T10:20:38.165534+00:00
---

## 标签
知识图谱，PCST 算法，Steiner Tree，路径预测，图谱补全，图算法

## 摘要
PCST（Prize-Collecting Steiner Tree）是一种图算法，用于在图中找到连接重要节点的最优子图，同时最小化连接成本并最大化收集的"prize"。该算法可应用于知识图谱的关系路径预测和补全任务，通过找到实体间的最低成本路径来推断潜在关系。

## 关键概念
- PCST: Prize-Collecting Steiner Tree，一种寻找最优连接子图的算法
- Prize Nodes: 图中需要连接的重要节点，带有表示重要性的 prize 值
- 知识图谱补全: 通过已有关系路径预测缺失的实体或关系


- 关系路径预测: 利用图结构推断实体间的潜在关联
- 连接成本: 边的权重通常反映关系的置信度或强度

## 关联笔记
- 01KJBZ9VVMKVM4Y1MX1TXPMJ7W.md: 同样涉及知识图谱补全，但采用 TransE 模型而非 PCST 算法
- 01KJBZCR153A9Q5TVFRCWHK44P.md: 探讨 ToG 框架在知识图谱上进行多跳推理的方法
- 01KJBZJCR5P60FXY3BH47TAFH4.md: 介绍 HippoRAG 如何利用 PageRank 算法在知识图谱中检索信息
## 标签
知识图谱，PCST 算法，Steiner Tree，路径预测，图谱补全，图算法

## 摘要
PCST（Prize-Collecting Steiner Tree）是一种图算法，用于在图中找到连接重要节点的最优子图，同时最小化连接成本并最大化收集的"prize"。该算法可应用于知识图谱的关系路径预测和补全任务，通过找到实体间的最低成本路径来推断潜在关系。

## 关键概念
- PCST: Prize-Collecting Steiner Tree，一种寻找最优连接子图的算法
- Prize Nodes: 图中需要连接的重要节点，带有表示重要性的 prize 值
- 知识图谱补全: 通过已有关系路径预测缺失的实体或关系
- 关系路径预测: 利用图结构推断实体间的潜在关联
- 连接成本: 边的权重，通常基于关系置信度确定

## 关联笔记
- 01KJBZ9VVMKVM4Y1MX1TXPMJ7W.md: 同样涉及知识图谱补全，但使用 TransE 模型进行预测
- 01KJBZCR153A9Q5TVFRCWHK44P.md: 讨论 ToG 算法在知识图谱上进行多跳推理路径规划
- 01KJBZJCR5P60FXY3BH47TAFH4.md: 介绍 HippoRAG 使用 PageRank 算法在知识图谱中进行知识发现
