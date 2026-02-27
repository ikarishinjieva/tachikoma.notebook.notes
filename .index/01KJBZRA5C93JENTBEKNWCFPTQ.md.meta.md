---
note: 01KJBZRA5C93JENTBEKNWCFPTQ.md
title: 20250205 - 阅读论文*: SELF-CONSISTENCY PREFERENCE OPTIMIZATION
indexed_at: 2026-02-27T11:32:13.038087+00:00
---

用户要求我分析这篇关于论文阅读的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20250205 - 阅读论文*: SELF-CONSISTENCY PREFERENCE OPTIMIZATION
2. 笔记内容主要关于：
   - 论文的核心观察：模型对问题多个变种答案中，一致性越高的部分正确率越高
   - 基于这个观察，用 DPO 训练，生成一致性高和低的答案形成训练对
   - 详细解释了 NLL（负对数似然）在 DPO 训练中的作用
   - NLL 用于保持自然语言流畅性

关键概念：
- SELF-CONSISTENCY PREFERENCE OPTIMIZATION（自一致性偏好优化）
- DPO（Direct Preference Optimization）
- NLL（Negative Log-Likelihood，负对数似然）
- 一致性

关联笔记分析：
查看提供的笔记列表，主要是关于：
- MySQL/数据库相关
- Clickhouse 相关
- 系统性能调试
- Java/连接池等

这篇笔记是关于机器学习/LLM 训练的论文阅读，与列表中的数据库、系统调试类笔记没有明显关联。

现在生成索引：

标签：LLM, DPO, 偏好优化, 自一致性, NLL, 论文阅读

摘要：这篇笔记记录了一篇关于自一致性偏好优化（Self-Consistency Preference Optimization）的论文阅读心得。

论文利用答案一致性构建 DPO 训练数据，并阐述了 NLL 在维持生成文本流畅性中的关键作用。

关键概念：
- SELF-CONSISTENCY PREFERENCE OPTIMIZATION: 基于答案一致性构建偏好优化训练对的论文方法
- DPO: Direct Preference Optimization，直接偏好优化算法
- NLL: 负对数似然，用于保持生成文本的自然语言流畅性
- 一致性：模型对相似问题回答中一致的部分更可能正确

关联笔记：无（这篇是机器学习/LLM 论文，其他笔记主要是数据库和系统调试）## 标签
LLM, DPO, 偏好优化, 自一致性, NLL, 论文阅读

## 摘要
这篇笔记记录了 SELF-CONSISTENCY PREFERENCE OPTIMIZATION 论文的核心思路：利用模型对相似问题回答的一致性高低构建 DPO 训练对。笔记详细解释了 NLL 在 DPO 训练中保持生成文本自然语言流畅性的作用机制。

## 关键概念
- SELF-CONSISTENCY PREFERENCE OPTIMIZATION: 基于答案一致性构建偏好优化训练对的论文方法
- DPO: Direct Preference Optimization，直接偏好优化算法
- NLL: 负对数似然，用于保持生成文本的语法正确和语义通顺
- 一致性: 模型对相似问题回答中一致的部分正确率更高

## 关联笔记
无
