---
note: 01KJBZAQZ6CX2KZQ4FC02BPFXS.md
title: 20240718 - ChatDBA: 如何使用知识类的文档
indexed_at: 2026-02-27T11:07:29.104204+00:00
---

用户希望我分析这篇关于 ChatDBA 知识文档和知识图谱的笔记，生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：关于 ChatDBA 项目中知识文档的整理和知识图谱的构建与检索方法

2. **核心内容**：
   - 知识文档的分类和存储结构
   - 朴素向量检索方法（bge-m3 + haystack）
   - 图方法（Neo4j + 知识图谱）
   - 测试案例和分析
   - 改进思路（参考 ConceptNet 等）

3. **关键概念**：
   - 知识图谱
   - Neo4j
   - 向量化检索
   - ConceptNet
   - bge-m3

4. **关联笔记**：
   查看提供的笔记列表，主要涉及：
   - MySQL 相关（刷脏页、performance_schema、临时表等）
   - Clickhouse 相关
   - 其他数据库技术

   这篇笔记主要讲知识图谱构建和检索方法，与 MySQL 技术细节的笔记关联度较低。但有一些可能相关：
   - 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 20210525 - 临时磁盘表...（与笔记中测试案例"临时表"有关联）
   - 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用（都是数据库相关知识）
   
   不过这些关联并不强，因为笔记重点在于知识图谱的方法论，而非具体的数据库技术细节。## 标签
知识图谱，Neo4j，向量化检索，ChatDBA，ConceptNet，文档管理

## 摘要
笔记整理了 ChatDBA 项目中知识文档的分类存储结构，对比了朴素向量检索与知识图谱两种检索方法。通过测试案例发现图谱查询命中率低，提出需统一图谱构建规范与查询规范，并参考 ConceptNet 等 NLP 库设计关系词体系。

## 关键概念
- 知识图谱：使用 Neo4j 存储 MySQL 手册生成的节点关系图谱
- 向量化检索：使用 bge-m3 模型对文档进行向量化，在 haystack 中检索
- ConceptNet：常识知识图谱，提供 34 个标准关系词用于语义查询
- 文档分类：将知识文档分为 old_data、error_doc、other_doc 三类管理

## 关联笔记
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 均涉及 MySQL 临时表相关问题的研究与观测
