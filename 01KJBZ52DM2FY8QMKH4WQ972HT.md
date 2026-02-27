---
title: 20240104 - 对langchain的ConversationEntityMemory的分析
confluence_page_id: 2589956
created_at: 2024-01-04T09:18:13+00:00
updated_at: 2024-01-04T09:18:13+00:00
---

# 主要作用

对记录的信息, 进行entity分析, 以及以entity为维度进行总结

# 流程

  - load_memory_variables: 分析对话的输入 (在chain.prep_inputs中调用)
    - 用LLM对输入进行实体分析, 抽取对话信息中的实体
    - 在内存中, 查询实体的历史摘要, 并返回历史摘要
  - save_context: 保存对话, 包括输入和输出 (在chain.prep_outputs中调用)
    - 对于当前对话历史, 对于load_memory_variables抽取的entity, 保存到对应entity的总结中
