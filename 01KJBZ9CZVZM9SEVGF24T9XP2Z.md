---
title: 20240520 - 阅读论文: XRICL: Cross-lingual Retrieval-Augmented In-Context Learning for Cross-lingual Text-to-SQL Semantic Par
confluence_page_id: 2949470
created_at: 2024-05-20T08:58:26+00:00
updated_at: 2024-05-20T09:36:38+00:00
---

# 主要内容

XRICL主要解决跨语言的Text-SQL问题, 其类似一个标准的RAG流程, 主要创新思路是: 

  1. 使用蒸馏训练, 训练了reranking模型
  2. 将翻译任务和生成任务, 同时写进few-shot提示词, 完成了两个任务的融合

# 蒸馏训练

用LLM做排序, 让reranking模型的KL散度逼近LLM

# 将翻译任务和生成任务, 同时写进few-shot提示词

![image2024-5-20 17:36:33.png](/assets/01KJBZ9CZVZM9SEVGF24T9XP2Z/image2024-5-20%2017%3A36%3A33.png)
