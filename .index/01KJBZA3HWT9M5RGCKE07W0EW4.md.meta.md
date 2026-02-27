---
note: 01KJBZA3HWT9M5RGCKE07W0EW4.md
title: 20240614 - 阅读论文: RepoCoder: Repository-Level Code Completion Through Iterative Retrieval and Generation
indexed_at: 2026-02-27T11:00:40.088582+00:00
---

用户要求我分析一篇关于 RepoCoder 论文的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记标题：20240614 - 阅读论文: RepoCoder: Repository-Level Code Completion Through Iterative Retrieval and Generation

2. 笔记内容核心：
   - 论文描述通过迭代方法对代码进行补全
   - 步骤：代码切片、RAG 召回相似代码片段、迭代重叠处理
   - 评价：没有太大创新，只是代码切片并迭代处理

3. 标签应该包括：代码补全、RAG、迭代检索、代码切片等

4. 摘要需要 2-3 句话概括

5. 关键概念：代码切片、RAG 召回、迭代处理等

6. 关联笔记：查看其他笔记列表，主要是关于 MySQL、Clickhouse、数据库、性能分析等内容，与这篇代码补全论文没有明显关联

让我生成输出：## 标签
代码补全，RAG，迭代检索，代码切片，论文阅读

## 摘要
论文提出 RepoCoder，通过迭代方式对代码仓库级别的代码进行补全。核心方法是分片处理代码，每片通过 RAG 召回相似代码辅助补全，下一迭代可复用上一迭代的优化结果。作者认为该方法创新性有限，本质是代码切片加迭代处理。

## 关键概念
- 代码切片：将代码分成多个片段，迭代处理，允许片段间重叠
- RAG 召回：检索增强生成，召回相似代码片段辅助补全
- 迭代优化：后续迭代可使用前序迭代优化后的结果作为已知片段

## 关联笔记
无
