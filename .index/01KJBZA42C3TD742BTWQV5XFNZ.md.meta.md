---
note: 01KJBZA42C3TD742BTWQV5XFNZ.md
title: 20240615 - 阅读论文: TIARA: Multi-grained Retrieval for Robust Question Answering over Large Knowledge Bases
indexed_at: 2026-03-05T10:11:10.312031+00:00
---

## 摘要
TIARA 论文提出多粒度检索方法，用于在大规模知识库上进行鲁棒的问答。通过实体识别、逻辑查找、Schema 检索三阶段，结合解码约束减少 SQL 生成错误。

## 关键概念
- Entity Retrieval: 从问题中检测提及、生成候选实体并进行消歧的三阶段实体识别流程
- Schema Retrieval: 独立查找与问题相关的表名和表连接关系的步骤
- Decoding Constraints: 在生成时限制模型仅在检索到的 Schema 范围内构建 SQL，减少伪造
- Exemplary Logical From Retrieval: 从实体触发，在训练数据库中查找两跳内的所有关系

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述笔记中引用了此 TIARA 论文作为相关工作
