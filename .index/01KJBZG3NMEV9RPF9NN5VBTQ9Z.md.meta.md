---
note: 01KJBZG3NMEV9RPF9NN5VBTQ9Z.md
title: 20241006 - 阅读论文*: RAGCHECKER: A Fine-grained Framework for Diagnosing Retrieval-Augmented Generation
indexed_at: 2026-03-05T10:45:57.845548+00:00
---

## 摘要
RAGCHECKER 是一个在 claim 级别评估 RAG 系统质量的细粒度框架，分别从整体、检索器和生成器三个层面进行诊断。该框架提供多项指标（如 Faithfulness、Noise Sensitivity、Hallucination 等）帮助定位问题来源，并基于 8 个 RAG 系统在 10 个数据集上的实验提出改进建议。

## 关键概念
- Claim Recall: 参考答案中的 claim 有多少被检索到的文本块覆盖
- Context Precision: 检索到的文本块中与参考答案相关的比例
- Faithfulness: 回复中的 claim 能在检索文本中找到依据的比例
- Noise Sensitivity: 回复中错误 claim 来自相关或不相关文本块的程度
- Context Utilization: 检索到的相关信息有多少被用于生成回复

## 关联笔记
- 01KJBZRDJC48S1985E9C14H3W2.md: 同一篇 RAGCHECKER 论文的另一份笔记，包含评估指标的图解说明
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 论文笔记，同样关注 RAG 系统中检索与生成的协同优化
- 01KJBZG9HE3MZ0312PBHKXC0M7.md: Speculative RAG 论文笔记，涉及使用验证器评估 RAG 生成质量
