---
title: 20240504 - 阅读论文: Prompt-RAG: Pioneering Vector Embedding-Free Retrieval-Augmented Generation in Niche Domains, Exemplifi
confluence_page_id: 2949324
created_at: 2024-05-04T15:51:38+00:00
updated_at: 2024-05-04T15:51:38+00:00
---

论文: Prompt-RAG: Pioneering Vector Embedding-Free Retrieval-Augmented Generation in Niche Domains, Exemplified by Korean Medicine

主要解决的问题: 在医疗领域, 基于Embedding的召回 不能理解一些医疗特质的微小差异

主要思路: 不通过Embedding召回, 而是通过查询目录, 找到相关文章

![image2024-5-4 23:29:2.png](/assets/01KJBZ86943BA9NRAA50KQKBF7/image2024-5-4%2023%3A29%3A2.png)

主要步骤: 

  1. 为文档生成目录(或主要内容)
  2. 检索时, 检索目录并召回相关文档
