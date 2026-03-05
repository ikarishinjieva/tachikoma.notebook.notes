---
note: 01KJBZ9NTPHXDJ944BG0R43D0S.md
title: 20240607 - 阅读论文*: FILCO: Learning to Filter Context for Retrieval-Augmented Generation
indexed_at: 2026-03-05T10:00:27.689200+00:00
---

## 标签
RAG, 上下文过滤，信息检索，句子级过滤，训练数据合成，语义抽取

## 摘要
FILCO 提出训练一个上下文过滤模型，对 RAG 召回文档进行句子级过滤以抽取关键信息。通过 STRINC、LEXICAL、CXMI 三种算法从 (问题，文档，答案) 数据生成训练样本，解决缺少句子级标注数据的问题。

## 关键概念
- STRINC: 判断段落是否包含生成输出，适用于答案直接出现的提取式问答
- LEXICAL: 计算文本片段与问题/答案的 unigram 重叠程度选择上下文
- CXMI: 计算提供上下文时生成正确输出的概率增加程度，适用于多步推理任务
- Oracle 上下文: 通过算法生成的理想过滤结果，作为模型训练的监督信号

## 关联笔记
- 01KJBZG2VSA818F7DC1JG7HF8X.md: 同样使用过滤器组件处理检索文档，但用于多跳问答的迭代检索场景
- 01KJBZG9HE3MZ0312PBHKXC0M7.md: 同样解决 RAG 召回文档过多的问题，采用聚类采样而非句子级过滤
