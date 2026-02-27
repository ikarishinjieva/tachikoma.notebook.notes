---
title: 20240609: 阅读论文*: GENREAD: GENERATE RATHER THAN RETRIEVE: LARGE LANGUAGE MODELS ARE STRONG CONTEXT GENERATORS
confluence_page_id: 2949613
created_at: 2024-06-09T03:04:03+00:00
updated_at: 2024-06-09T03:04:03+00:00
---

# 主要思路

面对一个问题, 不通过召回文档来获取知识, 而是通过不同的提示词从大模型中抽取知识文档, 来作为召回文档进行生成.

  - 提出了一种基于聚类的提示语方法，通过对文档进行聚类并从不同聚类中采样问题-文档对作为上下文示例，引导大型语言模型生成覆盖不同视角的文档。

这样的好处: 

  - 可以利用大模型内的知识, 来组织 召回的文档和问题 之间的逻辑
  - 大模型内的知识, 噪音比较小
