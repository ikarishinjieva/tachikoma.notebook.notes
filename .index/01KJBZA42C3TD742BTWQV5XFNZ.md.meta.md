---
note: 01KJBZA42C3TD742BTWQV5XFNZ.md
title: 20240615 - 阅读论文: TIARA: Multi-grained Retrieval for Robust Question Answering over Large Knowledge Bases
indexed_at: 2026-02-27T11:00:59.856077+00:00
---

用户要求我分析一篇关于论文阅读笔记，生成索引元数据。让我仔细分析笔记内容：

1. 笔记标题：20240615 - 阅读论文：TIARA: Multi-grained Retrieval for Robust Question Answering over Large Knowledge Bases

2. 笔记内容：关于 SQL 生成的论文，涉及：
   - 实体识别（Entity Retrieval）
   - 查找形式逻辑（Exemplary Logical From Retrieval）
   - 查找 Schema（Schema Retrieval）
   - 生成（带解码约束）

3. 其他笔记列表：大部分是关于 MySQL、Clickhouse、数据库相关的笔记，还有一些系统性能相关的。

让我分析关联：
- 这篇笔记是关于 SQL 生成/Text-to-SQL 的论文
- 其他笔记中，01KJBYDD09MGYR4GA0C4KB7P9F.md 提到 "TODO: Advanced Join Strategies for Large-Scale Distributed Computation" 也是论文/学习类笔记
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: SQLLancer 学习 - 这个可能和 SQL 相关
- 其他大部分是数据库运维、性能调优相关

关联度分析：
- SQLLancer 学习 可能有关联，因为都涉及 SQL
- 但 TIARA 是 Text-to-SQL/Question Answering 的研究，而其他笔记多是数据库运维层面


- 实际上关联度都不高

现在生成索引：

标签：Text-to-SQL, 实体识别, Schema 检索, SQL 生成, 知识图谱, 问答系统

摘要：这篇论文提出 TIARA 框架，通过多粒度检索解决大规模知识库上的问答问题。核心方法包括实体识别、逻辑形式检索和 Schema 检索，最后在解码阶段施加约束以生成准确的 SQL。

关键概念涉及实体消歧、两跳关系查找、解码约束和提及检测等机制。

关联笔记方面，SQLLancer 学习与 SQL 处理相关，而分布式计算策略的笔记可能涉及类似的 join 操作。不过这些关联度有限，其他笔记主要集中在数据库运维和性能调优，与 Text-to-SQL 研究方向差异较大。## 标签
Text-to-SQL, 实体识别, Schema 检索, SQL 生成, 知识图谱问答, 解码约束

## 摘要
论文 TIARA 提出多粒度检索方法，解决大规模知识库上的鲁棒问答问题。核心思路是通过实体识别、逻辑形式检索、Schema 检索三个独立模块，为 SQL 生成提供结构化约束，减少模型伪造。

## 关键概念
- Entity Retrieval: 从问题中抽取实体并映射到知识库主键的三阶段流程
- Exemplary Logical Form Retrieval: 从实体出发查找两跳内所有关系
- Schema Retrieval: 独立查找与问题相关的表名和表连接关系
- Decoding Constraints: 生成时限制模型仅在检索到的 Schema 范围内构建 SQL

## 关联笔记
- 01KJBYDAK7MGW5X6PQPXNF4BQ3.md: 同为 SQL 工具/技术学习类笔记
- 01KJBYDD09MGYR4GA0C4KB7P9F.md: 同为论文阅读 TODO 类笔记，涉及 Join 策略
