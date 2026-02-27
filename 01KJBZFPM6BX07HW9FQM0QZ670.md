---
title: 20240922 - 分析 dsRAG 项目 (对chunking的创新)
confluence_page_id: 3342466
created_at: 2024-09-22T14:27:39+00:00
updated_at: 2024-09-22T14:59:20+00:00
---

# 目标

使用了三种技术, 需要分析其流程

  - AutoContext 自动上下文
  - Semantic sectioning 语义切片
  - Relevant Segment Extraction 相关区段提取

# 添加文档的流程 - add_document

<https://github.com/D-Star-AI/dsRAG/blob/main/dsrag/knowledge_base.py#L279>

  1. 进行语义切片 semantic_sectioning, 否则将整个文档作为一个切片
  2. 对整篇文档生成标题: auto_context.get_document_title
  3. 对整篇文档生成摘要: auto_context.get_document_summary
  4. 对于每一个语义切片:
     1. 获取切片的摘要: auto_context.get_section_summary
     2. 将切片拆成chunk: split_into_chunks
  5. 对于每一个chunk, 增加描述头: get_chunk_header

# 查询文档的流程 - search
