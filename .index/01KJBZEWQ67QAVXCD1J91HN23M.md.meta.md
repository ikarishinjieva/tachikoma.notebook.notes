---
note: 01KJBZEWQ67QAVXCD1J91HN23M.md
title: 20240826 - 阅读论文: Rethinking the Role of Demonstrations: What Makes In-Context Learning Work?
indexed_at: 2026-02-27T11:14:12.015739+00:00
---

用户要求我分析这篇关于机器学习论文的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 In-Context Learning（语境学习）的论文笔记
2. 主要研究 demonstrations（样例）在 in-context learning 中的作用
3. 核心发现：标签可以是随机的，模型仍能工作
4. 关键因素：输入文本分布、标签空间、demonstrations 格式

现在我需要生成：
1. 标签（3-6 个关键词）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

看其他笔记列表，都是关于数据库、MySQL、Clickhouse、系统性能等技术内容，与这篇机器学习论文没有直接关联。所以关联笔记应该是「无」。## 标签
In-Context Learning, 大语言模型, 样例学习, 零样本学习, 预训练模型, 机器学习

## 摘要
论文研究了 In-Context Learning 中 demonstrations 的作用机制，发现模型性能对输入 - 标签映射关系不敏感，即使使用随机标签也能取得较好效果。关键影响因素是输入文本分布、标签空间和 demonstrations 格式，而非具体标签正确性。

## 关键概念
- In-Context Learning: 模型通过少量输入 - 标签对进行推理并完成新任务的学习方式
- Zero-shot Learning: 不提供任何样例，仅靠模型预训练知识完成任务
- Demonstrations: 语境学习中提供给模型的输入 - 标签配对样例
- 随机标签方法: 用随机而非正确标签配对输入作为 demonstrations 的低成本方案

## 关联笔记
无
