---
note: 01KJBZG9HE3MZ0312PBHKXC0M7.md
title: 20241007 - 阅读论文*: Speculative RAG: Enhancing Retrieval Augmented Generation through Drafting
indexed_at: 2026-02-27T11:17:46.889514+00:00
---

用户要求我分析一篇关于 Speculative RAG 论文的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 论文主题：Speculative RAG - 增强检索增强生成通过草稿机制
2. 核心问题：RAG 过程中过多召回文档导致 LLM 处理上下文成本过高
3. 方法：聚类+采样文档子集，用 Drafter 生成草稿，Verifier 验证选择最佳
4. Drafter 训练：从大模型蒸馏推理依据，然后指令微调

需要生成：
- 标签（3-6 个）
- 摘要（2-3 句话）
- 关键概念（3-5 个）
- 关联笔记（0-3 篇）

看关联笔记列表，都是关于数据库、MySQL、Clickhouse、系统性能等主题，与这篇 RAG/LLM 论文笔记没有明显关联。

让我生成输出：## 标签
RAG, 大语言模型, 检索增强生成, 文档采样, 模型蒸馏, 推理优化

## 摘要
该论文提出 Speculative RAG 方法，通过聚类采样将召回文档分成多个子集，并行生成答案草稿后择优，降低 LLM 处理长上下文的成本。Drafter 小模型通过蒸馏大模型的推理依据进行微调，Verifier 大模型负责评估草稿置信度。

## 关键概念
- Speculative RAG: 通过多子集草稿并择优的检索增强生成优化方法
- Drafter: 经蒸馏微调的小模型，负责并行生成答案草稿和推理依据
- Verifier: 大模型验证器，评估草稿置信度并选择最佳答案
- 多视角采样: 对召回文档聚类后从每类采样，减少冗余增加多样性
- 推理依据蒸馏: 用大模型生成解释，微调小模型学习推理能力

## 关联笔记
无
