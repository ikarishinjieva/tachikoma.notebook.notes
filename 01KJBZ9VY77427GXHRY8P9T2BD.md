---
title: 20240612 - 阅读论文: Ring: Repair Is Nearly Generation: Multilingual Program Repair with LLMs
confluence_page_id: 2949659
created_at: 2024-06-12T07:35:52+00:00
updated_at: 2024-06-12T07:35:52+00:00
---

# 主要思路

Ring的目标是修复代码报错, 类似于一个RAG过程: 

  - 检索: Ring通过错误代码进行检索
  - 生成
  - 排序选择: 排序的依据, 使用LLM生成的修补代码中, 每一个token的概率的对数和 (也就是LLM对哪一个修复代码片段更有信心), 选择对数和最高的片段

创新主要在检索(使用错误代码检索, 更专注)和排序(使用概率的对数和) 环节
