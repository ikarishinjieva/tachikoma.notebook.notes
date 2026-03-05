---
note: 01KJBZ5PKDX5D2ZCWN2N2BK8TQ.md
title: 20240208 - 阅读论文: RADIT: RA-DIT: RETRIEVAL-AUGMENTED DUAL INSTRUCTION TUNING
indexed_at: 2026-03-05T09:31:57.562319+00:00
---

## 摘要
RA-DIT 是一种轻量级微调方法，通过双阶段指令微调将 LLM 与检索能力结合：第一阶段微调语言模型以更好利用检索信息，第二阶段微调检索器以返回更符合 LM 偏好的结果。RA-DIT 65B 在知识密集型零样本和少样本基准测试中取得 SOTA 性能。

## 关键概念
- RA-DIT: Retrieval-Augmented Dual Instruction Tuning，检索增强双指令微调方法
- LSR (LM-Supervised Retrieval): 利用语言模型生成正确答案的概率来监督检索器微调
- 双编码器检索器: 文档编码器和查询编码器分别映射到嵌入空间，基于相似度检索
- 并行上下文检索增强: 将多个检索文本块单独添加到提示中并行计算，输出按相关性加权混合

## 关联笔记
- 01KJBZ59V9KQ9WBC9QH5ZP426S.md: REPLUG 论文同样使用 LSR 训练方法，是 RA-DIT 的技术基础
- 01KJBZ5MJTQ4XWXMBSZEB3QWH0.md: SELF-RAG 也是检索增强与自我反思结合的框架，可对比学习
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文，提供检索增强生成的整体技术背景
