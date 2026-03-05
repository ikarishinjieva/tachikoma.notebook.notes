---
note: 01KJBZ9FCKE73X8JQ26QF3N9BR.md
title: 20240601 - 阅读论文*: A fine-tuning enhanced rag system with quantized influence measure as ai judge
indexed_at: 2026-03-05T09:57:32.430484+00:00
---

## 摘要
论文提出双路答案生成系统，通过向量库召回和微调 LLM 并行生成两路答案，利用各自优势（向量库知识丰富但缺逻辑，LLM 逻辑灵活但知识有限）。使用量化影响度量 (QIM) 评估体系从双路答案中选择或合并生成最终答案，QIM 可通过反馈训练增强效果。

## 关键概念
- 双路生成: 向量库召回与微调 LLM 并行生成答案，互补知识与逻辑
- QIM(量化影响度量): 评估双路答案并选择/合并的评估体系，类似 reranking
- 反馈训练: 用于增强 QIM 评估效果的训练方法

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述笔记，在 Re-ranking 章节引用本论文的 QIM 技术
- 01KJBZWKEDHZEEK4TMX2V4XZ12.md: ChatDoctor 笔记，同样讨论微调+RAG 在医疗领域的精细化应用
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 笔记，讨论 RAG 与 MoE 结合的检索增强方案
