---
note: 01KJBZ86943BA9NRAA50KQKBF7.md
title: 20240504 - 阅读论文: Prompt-RAG: Pioneering Vector Embedding-Free Retrieval-Augmented Generation in Niche Domains, Exemplifi
indexed_at: 2026-03-05T09:46:53.057479+00:00
---

## 标签
RAG, 信息检索, 医疗领域, 论文解读, 目录索引, Embedding-Free

## 摘要
论文提出 Prompt-RAG 方法，解决医疗领域基于 Embedding 的召回无法理解专业术语微小差异的问题。核心思路是不通过向量相似度检索，而是为文档生成目录，检索时查询目录来召回相关文档。

## 关键概念
- Prompt-RAG: 无需向量嵌入的检索增强生成方法，通过目录查询实现文档检索
- Embedding-Free 检索: 不依赖向量相似度匹配，避免语义嵌入在专业领域的局限性
- 目录索引: 为文档生成结构化目录或主要内容摘要，作为检索的中间层

## 关联笔记
- 01KJBZWKEDHZEEK4TMX2V4XZ12.md: ChatDoctor 论文笔记，同样是医疗领域的 RAG 应用，采用关键词提取 + 结构化检索流程
- 01KJBZTFWATGMK1VW5NRVDQKMP.md: 包含基于检索的模型分类介绍，可作为检索方法的技术背景参考
