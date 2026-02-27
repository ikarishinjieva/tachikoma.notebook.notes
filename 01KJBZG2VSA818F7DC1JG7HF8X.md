---
title: 20241006 - 阅读论文*: EfficientRAG: Efficient Retriever for Multi-Hop Question Answering
confluence_page_id: 3342630
created_at: 2024-10-06T06:14:49+00:00
updated_at: 2024-10-06T06:25:41+00:00
---

# 论文目的

针对多跳问题的改进: 降低延迟/成本

# 框架方法

```
提出 EfficientRAG 框架：  EfficientRAG 包含两个轻量级组件：标签器/标注器（Labeler/Tagger）和过滤器（Filter）。
它们共享相同的模型结构，都是词级别的二元分类器，用于判断词语或文档块是否有用。
EfficientRAG 通过迭代生成新的查询进行检索，并同时保留最相关的检索信息，从而提高效率。
工作流程大致为：检索文档 ->  Labeler/Tagger 标记有用信息和文档相关性 ->  Filter 生成下一跳查询 -> 重复直至找到足够信息或达到最大迭代次数 ->  LLM 生成最终答案。

``` 

![image2024-10-6 14:10:46.png](/assets/01KJBZG2VSA818F7DC1JG7HF8X/image2024-10-6%2014%3A10%3A46.png)

批注:

  - EfficientRAG 和 迭代式RAG 的核心思路一样, 都是迭代式地 不断生成新的问题 并进行查询
  - 不同的是: EfficientRAG使用 Labeler + Tagger + Filter 替代了 迭代式RAG中 使用LLM生成"下一迭代的问题"的步骤
  - 其优点:
    - Labeler: 是个简单的二元分类问题: 哪些词是有用的. 这种模型即简单, 又具有泛化性
    - Tagger: 判断该文档块是否包含最终答案
    - Filter: 生成新的查询 (并过滤掉无用的信息)
    - Tagger和Filter使用小模型替代LLM, 主要原因是降低成本

# 有用的信息

  - EfficientRAG使用 Labeler + Tagger + Filter 替代了 迭代式RAG中 使用LLM生成"下一迭代的问题"的步骤
    - Labeler: 是个简单的二元分类问题: 哪些词是有用的. 这种模型即简单, 又具有泛化性
  - 论文中还包括了 用LLM进行训练数据合成 的描述
