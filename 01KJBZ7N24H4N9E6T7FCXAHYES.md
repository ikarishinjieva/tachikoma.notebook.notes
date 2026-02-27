---
title: 20240323 - 关于LLM的function call的理解
confluence_page_id: 2949222
created_at: 2024-03-22T16:18:08+00:00
updated_at: 2024-03-22T16:18:18+00:00
---

参考文档: <https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/function-calling?hl=zh-cn>

概要: 

  1. 通过提示词, 告诉LLM 存在一些function 
  2. LLM根据他的理解, 告诉Human: LLM需要调用哪个function, 参数是什么
  3. Human告诉LLM关于function的执行结果
  4. LLM进行后续处理
