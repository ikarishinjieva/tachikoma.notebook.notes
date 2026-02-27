---
title: 20241006 - 阅读论文: Contextual retrieval
confluence_page_id: 3342642
created_at: 2024-10-06T15:35:34+00:00
updated_at: 2024-10-06T15:35:34+00:00
---

<https://www.anthropic.com/news/contextual-retrieval>

# 有用的知识 

Embeddings+BM25 比单独使用Embeddings效果更好

  - BM25 用于匹配关键字

# 上下文处理

生成上下文摘要的提示词: 

将chunk和document都给到LLM, 来生成对 联结 有用的上下文摘要

```
<document> 
{{WHOLE_DOCUMENT}} 
</document> 
Here is the chunk we want to situate within the whole document 
<chunk> 
{{CHUNK_CONTENT}} 
</chunk> 
Please give a short succinct context to situate this chunk within the overall document for the purposes of improving search retrieval of the chunk. Answer only with the succinct context and nothing else.
```
