---
note: 01KJBZ5SXG26R34S13HPP05SYY.md
title: 20240209 - 阅读论文: ITER-RETGEN: Enhancing retrieval-augmented large language models with iterative retrieval-generation sy
indexed_at: 2026-03-05T09:32:57.666941+00:00
---

## 摘要
本笔记记录了 ITER-RETGEN 论文的核心内容，该方法通过迭代式的检索与生成协同来增强检索增强型大语言模型。笔记涵盖了论文结构、相关工作对比、方法细节、迭代步骤及终止条件等关键问答内容。

## 关键概念
- ITER-RETGEN: 一种迭代式检索与生成协同方法，利用前次生成输出作为上下文进行多轮检索优化
- 生成增强检索适应: 通过生成结果优化检索器，更好地捕捉问题与支持知识之间的语义联系
- 多跳问答: 需要多次检索和推理才能回答的复杂问题类型
- 性能收敛: 判断迭代终止的条件之一，当模型性能趋于稳定时可结束迭代

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文，在检索增强分类中引用了 ITER-RETGEN 方法
- 01KJBZ60RQ00K5XS0VZBK911N1.md: RRR 论文，同样针对检索增强 LLM 的查询重写优化方法
- 01KJBZ5MJTQ4XWXMBSZEB3QWH0.md: SELF-RAG 论文，通过自我反思机制实现检索与生成的另一种方案
