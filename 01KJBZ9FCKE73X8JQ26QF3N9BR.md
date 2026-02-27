---
title: 20240601 - 阅读论文*: A fine-tuning enhanced rag system with quantized influence measure as ai judge
confluence_page_id: 2949576
created_at: 2024-06-01T15:05:41+00:00
updated_at: 2024-06-01T16:26:23+00:00
---

# 主要逻辑

流程图: 

![image2024-6-1 22:49:40.png](/assets/01KJBZ9FCKE73X8JQ26QF3N9BR/image2024-6-1%2022%3A49%3A40.png)

本文主要创新: 

  1. 双路生成答案, 
     1. 通路1: 从向量库中召回
     2. 通路2: 从一个微调后的LLM生成答案
     3. 双路生成答案的好处是: 向量库召回的片段缺少逻辑, LLM的逻辑更灵活但知识片段比较少
  2. 通过一个评估体系QIM, 从双路答案中选择一个, 或者合并生成最终答案
     1. QIM可以通过 反馈训练 来增强效果
     2. 文中对QIM的具体算法没有提及
