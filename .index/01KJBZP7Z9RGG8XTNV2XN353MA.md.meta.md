---
note: 01KJBZP7Z9RGG8XTNV2XN353MA.md
title: 20241227 - 阅读论文: Fine-tuning large neural language models for biomedical natural language processing
indexed_at: 2026-03-05T11:08:30.153513+00:00
---

## 标签
生物医学 NLP, 模型微调, BERT, 微调稳定性, 分层自适应, PubMedBERT

## 摘要
该论文系统研究了生物医学 NLP 中大语言模型微调的稳定性问题，发现领域特定预训练（PubMedBERT）比通用预训练后适配的模型表现更优。提出了分层自适应方法（冻结底层、分层学习率衰减、重新初始化顶层）可有效提升微调稳定性，最佳策略取决于任务和预训练设置。

## 关键概念
- PubMedBERT: 使用 PubMed 摘要从头预训练的领域专用 BERT 模型
- 分层自适应方法: 冻结底层、分层学习率衰减、重新初始化顶层的微调策略组合
- BLURB: 生物医学 NLP 基准测试，用于评估模型在多种任务上的性能
- 微调不稳定性: 小数据集场景下模型性能波动大的问题

## 关联笔记
- 01KJBZZJ34XCYFH4C8WXDKJAMV.md: RS-LoRA 论文，同样研究微调稳定性问题，提出秩稳定缩放因子
- 01KJBZZHDRJBS1FEX3M60AZ305.md: DoRA 论文，研究 LoRA 改进方法，分析全量微调与参数高效微调的差距
- 01KJBZRQ6GE4B51ZFRHXA7TN44.md: SFT 与 RL 对比研究，探讨不同后训练方法的泛化与记忆特性
