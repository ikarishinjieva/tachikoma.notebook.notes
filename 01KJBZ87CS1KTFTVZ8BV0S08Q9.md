---
title: 20240504 - 阅读论文: QA-RAG
confluence_page_id: 2949328
created_at: 2024-05-04T16:17:04+00:00
updated_at: 2024-05-04T16:17:04+00:00
---

<https://mp.weixin.qq.com/s/Vq2Kbml2VBcKi8s4H1KXUw>

主要思路: 

  - 用一个微调后的LLM, 生成 fake answer
  - 将(fake answer + 原始query) 来召回文档

思路类似于HyDE, 区别在于HyDE使用了通用LLM, 这样会引入 杂音知识
