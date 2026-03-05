---
note: 01KJBZ5CS7RW5E8BST5VHXFD9A.md
title: 20240204 - 阅读论文: Self-mem: Retrieval-augmented text generation with self memory.
indexed_at: 2026-03-05T09:30:08.074618+00:00
---

## 摘要
Selfmem 是一种通过迭代使用检索增强生成器创建无限记忆池的框架，利用模型自身输出作为自记忆来改进生成过程。在神经机器翻译、文本摘要和对话生成任务上验证了有效性，展示了自记忆增强检索生成模型的潜力。

## 关键概念
- Selfmem: 利用自记忆改进文本生成的迭代式框架，突破固定语料库限制
- 检索增强生成器: 通过微调小模型或 few-shot LLM 生成候选文本
- 记忆选择器: 根据评估指标 (如 BLEU/ROUGE) 从候选池中选择最佳输出作为下一轮记忆
- 自记忆: 模型自身生成的输出作为后续生成的记忆输入，扩展记忆空间

## 关联笔记
- 01KJBZ59V9KQ9WBC9QH5ZP426S.md: REPLUG 论文笔记，同为检索增强生成方向研究，训练方法相似
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文，将 Selfmem 列为 RAG 相关研究
- 01KJBZ65Y2QFCFWKWQVM7T6NQ3.md: 提到 memory-selector 相当于对 reranking 的微调，与记忆选择器机制相关
