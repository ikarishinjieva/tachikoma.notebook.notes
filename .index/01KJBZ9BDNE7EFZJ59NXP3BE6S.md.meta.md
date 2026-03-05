---
note: 01KJBZ9BDNE7EFZJ59NXP3BE6S.md
title: 20240517 - 阅读论文: RAPTOR: RECURSIVE ABSTRACTIVE PROCESSING FOR TREE-ORGANIZED RETRIEVAL
indexed_at: 2026-03-05T09:54:27.121557+00:00
---

## 摘要
RAPTOR 是一种新型检索增强系统，通过递归嵌入、聚类和总结文本块构建多层次摘要树结构。系统支持树遍历和折叠树两种检索策略，能有效整合不同抽象级别的信息，在处理长篇文档复杂推理任务时优于传统检索方法。

## 关键概念
- 递归摘要树: 从下到上多层次构建的树状结构，叶节点为原始文本块，高层节点为聚类和摘要结果
- 文本块嵌入: 使用 SBERT 等编码器将约 100 词的文本块转换为向量表示
- 聚类总结: 使用 GMM 等算法将语义相似文本块聚集，再用 LLM 凝练主要信息
- 树遍历检索: 逐层遍历树结构，选择每层中与查询最相关的节点
- 折叠树检索: 将树中所有节点视为同一层处理，直接选择最相关节点

## 关联笔记
- 01KJBZG9HE3MZ0312PBHKXC0M7.md: Speculative RAG 同样使用聚类 + 采样方法处理召回文档，减少冗余并增加多样性
- 01KJBZQZQJR5F08D2AHTD88G9S.md: META-CHUNKING 研究文本分块策略，与 RAPTOR 的初始文本块分割相关
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 探索检索增强与推理结合，与 RAPTOR 同属 RAG 优化方向
