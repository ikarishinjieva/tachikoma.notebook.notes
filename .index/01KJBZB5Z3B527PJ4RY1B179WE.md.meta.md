---
note: 01KJBZB5Z3B527PJ4RY1B179WE.md
title: 20240725 - 知识图谱的关系抽取[2]
indexed_at: 2026-02-27T11:08:36.685476+00:00
---

用户要求我分析一篇关于知识图谱关系抽取的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：使用 LLM 构建知识图谱，特别是从 MySQL 文档中抽取实体并构建关系图
2. 核心内容：
   - 从 MySQL 文档中抽取实体（如 MySQL、子查询、存储函数等）
   - 使用 mermaid 构建关系图
   - 提出了查询问题（如何查询 MySQL signal 8 错误、临时表使用场景）
   - TODO 提到参考 LlamaIndex 的 Graph 模块和 CoE（KG-RAG 相关论文）

3. 标签应该包括：知识图谱、关系抽取、LLM、MySQL、KG-RAG 等

4. 关键概念：实体抽取、关系图、知识图谱构建等

5. 关联笔记：查看提供的笔记列表，大部分是 2021 年的 MySQL/数据库相关笔记，但这篇是 2024 年关于知识图谱的。可能关联的有：
   - 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 刷脏页相关（MySQL 内部机制）
   - 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究
   - 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 临时磁盘表相关（与笔记中提到的"临时表"问题有关联）
   
   但这些关联度都不高，因为这篇笔记主要是关于知识图谱构建方法，而不是 MySQL 具体问题。我可以选 1-2 个 MySQL 相关的作为潜在关联，或者写"无"。

现在生成输出。## 标签
知识图谱, 关系抽取, LLM, MySQL, KG-RAG, 实体抽取

## 摘要
探索使用 LLM 从技术文档中抽取实体并构建知识图谱关系图的方法。以 MySQL 官方文档为例，演示了实体抽取、mermaid 关系图生成的完整流程，并提出后续查询场景和 TODO 方向。

## 关键概念
- 实体抽取: 从文本中识别并提取领域内的关键概念作为搜索关键字
- 关系图: 使用 mermaid 将实体间的逻辑关系可视化为有向图
- KG-RAG: 知识图谱与检索增强生成结合的技术方案（CoE 论文方向）

## 关联笔记
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 都涉及 MySQL 临时表相关问题的研究
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同为 MySQL InnoDB 机制相关的技术探索笔记
