---
title: 20250131 - 阅读论文: Query优化综述: A Survey of Query Optimization in Large Language Models
confluence_page_id: 3344039
created_at: 2025-01-30T16:11:50+00:00
updated_at: 2025-01-30T16:11:50+00:00
---

中文分析: <https://mp.weixin.qq.com/s/XImZp0655W5Ee-RmkD9ZKg>

从其中截取了一些可能有用的方法: 

1\. Query Expansion（查询扩展）

让检索模型利用 LLM 生成知识集合扩展查询知识，同时 LLM 借助检索文档优化提示；

基于原始查询预测未来内容并检索信息，迭代优化；

挖掘语料库知识，用 LLMs 评估相关性确定关键句子扩展查询；

  
2\. Question Decomposition（问题分解）

用查询扩展模型生成多样查询，经重排选择更好检索结果；

用子方面探索器剖析查询，结合多方面检索器回答。

  
3\. Query Disambiguation（查询消歧）

引入基于自然语言的演绎推理格式，分解推理过程为子过程，增强推理严谨性；

用 “rewrite-then-edit” 框架让 LLMs 改写和编辑查询消除歧义；

利用 LLMs 的 NLP 能力（如解决共指关系、扩展上下文）减少对话历史歧义，通过多种方式将优化后的对话历史融入框架。

  
4\. Query Abstraction（查询抽象）

将通用 CoT 推理抽象为含抽象变量推理链，借助领域工具解决查询；

用抽象框架构建推理过程，集成不同层次抽象；
