---
title: 20240504 - ChatDBA: 预处理文档并生成高质量Plan, 需要进行论文查找
confluence_page_id: 2949320
created_at: 2024-05-04T15:30:49+00:00
updated_at: 2024-05-04T16:27:48+00:00
---

# 背景

ChatDBA第一期完成, 发现的问题是: 必须生成高质量的Plan, 后面的过程才能高质量且稳定. 

第一期的流程中: 

  1. 从文档中抽取了 "需要采集的信息"
  2. 将其 按照人类的分类 进行了排序
  3. 生成Plan时, 使用了步骤2的信息顺序, 但允许调序
     - 问题是: 如何使用这些信息, 主要来自于 LLM内置的知识 (召回的文档片段中, 仅有少量的 多信息之间的联系)
     - 这样生成的Plan的质量非常不稳定
  4. 为了平抑这种质量不稳定, 第一期生成了三个Plan, 选择一个最优的

在第二期, 我们需要一些方法, 将文档改写成排查Plan, 这样在生成Plan的时候, 这部分信息可以被有效利用.

# 论文列表

  - [20240504 - 阅读论文: Prompt-RAG: Pioneering Vector Embedding-Free Retrieval-Augmented Generation in Niche Domains, Exemplified by Korean Medicine]
    - 放弃Embedding, 预生成"目录" 进行文档召回
  - [20240504 - 阅读论文: QA-RAG]
    - 用一个微调后的LLM, 生成 fake answer, 将(fake answer + 原始query) 来召回文档
  - <https://www.semanticscholar.org/paper/Improving-Retrieval-Augmented-Open-Domain-with-Chen-Wang/bbf7233973fda41617f0603e91f7b292c6c01f13>
  - <https://www.semanticscholar.org/paper/Retrieval-Augmented-Generation-for-AI-Generated-A-Zhao-Zhang/ab15463babf98fffc6f683fe2026de0725b5e1a9>
