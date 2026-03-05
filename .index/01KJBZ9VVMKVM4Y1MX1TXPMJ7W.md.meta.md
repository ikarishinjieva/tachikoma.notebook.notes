---
note: 01KJBZ9VVMKVM4Y1MX1TXPMJ7W.md
title: 20240612 - 阅读论文: CBR-KBQA: Case-based reasoning for natural language queries over knowledge bases
indexed_at: 2026-03-05T10:05:06.816947+00:00
---

## 摘要
CBR-KBQA 是基于知识图谱的问答方法，采用案例推理三阶段：召回案例、复用案例、修订案例。在修订阶段使用 TransE 模型进行知识图谱补全，预测缺失的关系。

## 关键概念
- CBR (Case-Based Reasoning): 案例推理方法，通过检索和复用历史案例解决新问题
- TransE: 知识图谱嵌入模型，学习实体和关系的低维向量表示用于关系预测
- 知识图谱补全: 预测知识图谱中缺失的三元组关系
- KBQA (Knowledge Base Question Answering): 基于知识库的自然语言问答

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: 该笔记汇总了多篇 KBQA 相关论文，明确引用并总结了 CBR-KBQA 的核心思路
- 01KJBZ8DW4SBX4299F8N84SB35.md: ECBRF 论文同样使用 CBR 方法进行常识知识库补全，与本文主题相近
- 01KJBZANGZVY92HGNK5652A6G8.md: 介绍 PCST 算法在知识图谱补全中的应用，与本文 TransE 补全目标一致
