---
note: 01KJBZ59V9KQ9WBC9QH5ZP426S.md
title: 20240204 - 阅读论文: REPLUG: Retrieval-Augmented Black-Box Language Models
indexed_at: 2026-03-05T09:29:27.155126+00:00
---

## 摘要
笔记记录了 REPLUG 论文的核心方法：通过检索模块增强冻结的黑盒语言模型，采用文档加权集成的方式克服上下文长度限制。重点分析了 fastRAG 实现中 invocation_layer 的定制逻辑，包括 REPLUGLogitsProcessor 在 sample 阶段修订 token 权重的机制。

## 关键概念
- REPLUG: 检索增强黑盒语言模型，将语言模型视为黑盒，通过可调节检索模块增强
- 输入重构: 将检索文档与输入上下文拼接，采用加权平均集成多次传递的输出概率
- REPLUGLogitsProcessor: 根据文档相似度得分修订 logits 权重，影响采样过程
- LSR(Language Model Supervised Retrieval): 用语言模型困惑度作为监督信号训练检索器
- Invocation Layer: 组织 tokenizer + generate + decode 流程的最外层，REPLUG 在此注入修订逻辑

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文，在分类和增强方法中引用了 REPLUG
- 01KJBZ5PKDX5D2ZCWN2N2BK8TQ.md: RA-DIT 论文，采用类似的检索增强双指令微调方法
- 01KJBZ65Y2QFCFWKWQVM7T6NQ3.md: 调优笔记，提及 REPLUG LSR 原理但无实现细节
