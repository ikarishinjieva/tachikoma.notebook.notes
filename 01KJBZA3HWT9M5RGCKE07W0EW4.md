---
title: 20240614 - 阅读论文: RepoCoder: Repository-Level Code Completion Through Iterative Retrieval and Generation
confluence_page_id: 2949705
created_at: 2024-06-14T06:59:47+00:00
updated_at: 2024-06-14T07:00:34+00:00
---

主要思路: 

论文主要描述了通过迭代的方法, 对代码进行补全. 

具体步骤: 

  1. 在迭代中对代码进行切片 (每迭代处理一片, 允许和上一片有一部分重叠)
  2. 对每一个切片, 进行RAG召回相似代码片段, 进行补全
  3. 下个迭代的切片中, 重叠部分带有本次迭代优化后的结果

RepoCoder没有太大的创新, 只是进行代码切片并迭代处理, 下一迭代可以用上一迭代的结果作为已知片段, 进行继续的优化
