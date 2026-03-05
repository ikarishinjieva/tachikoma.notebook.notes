---
note: 01KJBZEZD6R9ARWYKZGV5DM5TJ.md
title: 20240826 - 阅读论文: FINETUNED LANGUAGE MODELS ARE ZERO-SHOT LEARNERS
indexed_at: 2026-03-05T10:39:30.824203+00:00
---

## 标签
Instruction Tuning, FLAN, 零样本学习, 大语言模型, 微调, 任务泛化

## 摘要
论文提出指导调整（Instruction Tuning）方法，通过在 60+ 个 NLP 任务上使用自然语言指令模板微调预训练模型，显著提升 LLM 在未见任务上的零样本学习能力。核心创新在于用自然语言指令桥接预训练模型与多种任务，使模型学会理解并执行多样化指令而非单纯数据增强。

## 关键概念
- Instruction Tuning（指导调整）: 使用自然语言指令模板在多任务混合数据集上微调预训练语言模型的方法
- 零样本学习: 模型在训练时未见过的任务上，仅凭自然语言指令描述即可正确执行任务的能力
- 指令模板: 为同一任务手工制作的多种自然语言表述形式，用于增强模型对任务的理解和适应能力
- FLAN 模型: 经过指导调整微调后的 LaMDA-PT 模型，在零样本任务上性能超越 GPT-3 等基准模型

## 关联笔记
- 01KJBZ5PKDX5D2ZCWN2N2BK8TQ.md: RA-DIT 论文同样涉及指令微调技术，但聚焦于检索增强的双阶段微调方法
- 01KJBZ5BY0F6897T3ZEDW3GA6H.md: UPRISE 论文同样关注零样本评估问题，通过提示检索而非微调来提升零样本性能
- 01KJBZG071EGFZEW1XATH1JV1T.md: REAP 论文讨论了零样本提示的局限性，与 FLAN 通过微调解决零样本能力的思路形成对比
