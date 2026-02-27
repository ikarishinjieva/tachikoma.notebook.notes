---
note: 01KJBZ0T32KHT2AAC9NBG5X8R9.md
title: 20230608 - Text-To-SQL 的测试基准集 和 UnifiedSKG框架
indexed_at: 2026-02-27T10:30:24.757660+00:00
---

用户要求我分析一篇关于 Text-To-SQL 测试基准集和 UnifiedSKG 框架的笔记，并生成索引元数据。我需要：

1. 提取 3-6 个标签
2. 写 2-3 句话的摘要
3. 列出 3-5 个关键概念
4. 从提供的笔记列表中找出 0-3 篇关联笔记

让我分析笔记内容：
- 主题是 Text-To-SQL 的测试基准集（DAMO-ConvAI/BIRD）和 UnifiedSKG 框架
- 涉及 SQL 数据集（cosql/sparc/spider/sql2text/wikisql）
- UnifiedSKG 是结构化知识处理框架，融合 21 个预处理任务
- 待处理问题是如何解决自定义 SQL 规则问题

关联笔记分析：
- 大多数笔记是关于 MySQL、Clickhouse、性能调优等
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 20210429 - SQLLancer 学习 - 这个与 SQL 相关
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 20210606 - 从 binlog 中分离 SQL, 进行统计 - 这个与 SQL 相关

其他笔记主要是数据库性能、刷盘、连接池等，与 Text-To-SQL 和 NLP 框架关联较弱。

现在开始生成元数据。## 标签
Text-To-SQL, 测试基准集, UnifiedSKG, 结构化知识, SQL 数据集, 大模型

## 摘要
介绍 DAMO-ConvAI 的 BIRD 测试基准集及其包含的多个 SQL 数据集（cosql/sparc/spider 等），用于测试 Text-To-SQL 准确度。同时介绍 UnifiedSKG 框架，该框架融合 21 个预处理任务，将结构化知识转为文本供大模型处理。

## 关键概念
- DAMO-ConvAI/BIRD: Text-To-SQL 准确度测试基准集
- UnifiedSKG: 结构化知识处理框架，将 SQL/表格等转为文本由大模型理解
- SKG (Structured Knowledge Grounding): 结构化知识（SQL/表格等）的表示与处理
- T5 模型: 可用于 Fine-tuning 的文本大模型
- SQL 数据集: 包含 cosql/sparc/spider/sql2text/wikisql 等标注数据集

## 关联笔记
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: SQLLancer 学习，同属 SQL 工具/技术学习方向
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 从 binlog 中分离 SQL 进行统计，涉及 SQL 处理与分析
