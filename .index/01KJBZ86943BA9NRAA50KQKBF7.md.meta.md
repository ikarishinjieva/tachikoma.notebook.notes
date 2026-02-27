---
note: 01KJBZ86943BA9NRAA50KQKBF7.md
title: 20240504 - 阅读论文: Prompt-RAG: Pioneering Vector Embedding-Free Retrieval-Augmented Generation in Niche Domains, Exemplifi
indexed_at: 2026-02-27T10:51:27.399303+00:00
---

用户要求我分析一篇关于 Prompt-RAG 论文的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记主题：关于 Prompt-RAG 论文的阅读笔记，这是一种在特定领域（如韩医）中不使用向量 Embedding 的 RAG 方法
2. 核心内容：
   - 解决的问题：医疗领域基于 Embedding 的召回不能理解医疗特质的微小差异
   - 主要思路：通过查询目录找到相关文章，而不是 Embedding 召回
   - 步骤：为文档生成目录，检索时检索目录并召回相关文档

3. 标签应该是：RAG, 信息检索，医疗领域，目录检索，Embedding-free 等

4. 关联笔记分析：
   - 浏览其他笔记列表，大部分是关于 MySQL、Clickhouse、数据库、系统性能等方面的笔记
   - 这篇笔记是关于 RAG/信息检索的，与其他数据库/系统类笔记关联度较低
   - 没有明显的关联笔记

让我按照要求的格式输出。## 标签
RAG, 信息检索, 医疗领域, 目录检索, Embedding-free, 论文阅读

## 摘要
该笔记记录了 Prompt-RAG 论文的核心内容，提出了一种在医疗等特定领域中不依赖向量 Embedding 的检索增强生成方法。通过为文档生成目录并在检索时查询目录来召回相关文档，解决了传统 Embedding 召回无法理解医疗特质微小差异的问题。

## 关键概念
- Prompt-RAG: 一种不使用向量 Embedding 的检索增强生成方法
- 目录检索: 通过查询文档目录而非向量相似度来召回相关文档
- Embedding-free: 不依赖向量嵌入的检索方式，适用于需要理解细微语义差异的领域

## 关联笔记
无
