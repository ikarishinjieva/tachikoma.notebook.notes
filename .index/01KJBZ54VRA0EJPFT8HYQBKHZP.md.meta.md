---
note: 01KJBZ54VRA0EJPFT8HYQBKHZP.md
title: 20240131 - 阅读论文: Enhancing Financial Sentiment Analysis via Retrieval Augmented Large Language Models
indexed_at: 2026-03-05T09:28:03.662793+00:00
---

## 摘要
论文提出一种针对金融情感分析的检索增强 LLMs 框架，通过指导微调模块和检索增强模块解决金融新闻简洁性和缺乏上下文的问题。该方法在多个金融情感分析基准测试上相比传统模型和其他 LLMs 实现了 15% 到 48% 的性能提升。

## 关键概念
- 检索增强 LLMs 框架: 结合指导微调模块和检索增强模块，从可靠外部来源检索上下文信息以增强情感分析准确性
- 指导调整模块: 使用专为金融情感分析设计的指导示例对 LLMs 进行微调，确保模型作为情感标签预测器表现良好
- 检索增强模块: 从新闻媒体、研究出版平台等可靠来源检索额外上下文信息，弥补金融新闻缺乏背景的问题
- 标准 RAG 区别: 该框架针对金融情感分析任务优化，LLMs 经过专门微调，而标准 RAG 用于语言生成任务且通常无专门微调

## 关联笔记
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 论文同样研究检索增强生成，使用反射标记和 MoE 架构提升复杂推理任务性能
- 01KJBZWKEDHZEEK4TMX2V4XZ12.md: ChatDoctor 针对医疗领域的微调+RAG 方案，同样对特定高风险领域进行深度优化
- 01KJBZRB8197X3KFWB1F72P7VG.md: RICHES 论文探索检索与序列生成交错的方法，在 LLM 思考过程中动态生成检索词
