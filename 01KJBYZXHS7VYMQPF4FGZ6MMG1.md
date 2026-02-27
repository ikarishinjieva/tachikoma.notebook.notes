---
title: 20230513 - 对langchain的MapReduceDocumentsChain机制分析
confluence_page_id: 2130918
created_at: 2023-05-13T09:24:37+00:00
updated_at: 2023-05-13T09:24:37+00:00
---

- _call 对 docs数组 调用 combine_docs
  - combine_docs (docs)
    - 对每个doc, 调用llm_chain, 采集结果 (数组)
    - 以 tokens_max 为限制, 将 docs 进行分组, 每个分组的文档的总长度不超过 tokens_max
      - 对每个分组, 调用_collapse_chain, 将分组内的多个文档 合并成 单个文档
      - 将_collapse_docs后的文档构成 新分组
      - 如果 新分组 总长度超过tokens_max, 则再次进行_collapse_docs
    - 对于最后一轮的 文档分组, 调用 combine_document_chain, 合成统一的文字
