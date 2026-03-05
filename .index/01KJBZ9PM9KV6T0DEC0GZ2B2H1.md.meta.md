---
note: 01KJBZ9PM9KV6T0DEC0GZ2B2H1.md
title: 20240609: 阅读论文*: GENREAD: GENERATE RATHER THAN RETRIEVE: LARGE LANGUAGE MODELS ARE STRONG CONTEXT GENERATORS
indexed_at: 2026-03-05T10:00:45.169708+00:00
---

## 摘要
提出 GENREAD 方法，通过提示词从大模型内部生成知识文档替代传统检索召回。利用基于聚类的提示语方法引导模型生成多视角文档，减少噪音并更好地组织知识逻辑。

## 关键概念
- GENREAD: 通过提示词从大模型生成知识文档而非检索外部文档的方法
- 聚类提示语: 对文档聚类后采样问题 - 文档对作为上下文示例，引导模型生成多视角内容
- 内部知识生成: 利用大模型参数化知识替代外部检索，降低噪音

## 关联笔记
- 01KJBZ9CGND48QFTK6R3W2QQ97.md: UniMS-RAG 论文笔记，采用传统检索增强方法，与 GENREAD 的生成而非检索思路形成对比
- 01KJBZ5PKDX5D2ZCWN2N2BK8TQ.md: RA-DIT 论文笔记，研究检索增强微调，同样基于检索范式可对比参考
