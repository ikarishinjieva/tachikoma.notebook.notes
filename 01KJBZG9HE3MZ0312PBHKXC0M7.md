---
title: 20241007 - 阅读论文*: Speculative RAG: Enhancing Retrieval Augmented Generation through Drafting
confluence_page_id: 3342646
created_at: 2024-10-07T08:45:18+00:00
updated_at: 2024-10-07T08:45:18+00:00
---

# 论文目的

RAG过程中, 过多的召回文档导致LLM处理上下文上的成本过高

# 论文方法

SPECULATIVE RAG的主要思路: 在召回的文档中, 进行聚类+采样 构成多个子集, 对每个子集生成草稿, 在判断草稿的质量择优

```
4. **SPECULATIVE RAG 的工作原理：**
    * **多视角采样：**将检索到的文档进行聚类，从每个聚类中采样一个文档组成子集，以减少冗余并增加多样性。
    * **专家 RAG 起草器 (Drafter)：**使用一个较小的、专门训练的 LLM (Drafter) 并行地处理多个文档子集，生成答案草稿和相应的推理依据。
    * **通才 RAG 验证器 (Verifier)：**使用一个较大的通用 LLM (Verifier) 根据推理依据评估答案草稿的置信度，并选择最佳草稿作为最终答案。
``` 

Drafter的训练, 相当于从大模型蒸馏出 推理依据E, 然后进行微调

```
1. **数据增强：** Drafter 的训练数据来源于各种指令遵循数据集，例如 Open-Instruct processed data 和一些知识密集型数据集。 对于每个指令-响应对 (Q, A)， 首先使用密集检索器 (例如 Contriever-MS MARCO) 检索相关的文档 D，然后使用一个强大的 LLM (例如 Gemini-Ultra) 生成解释响应 A 的推理依据 E。  这样就得到了增强后的训练数据 (Q, A, D, E)。  附录 A 中有更详细的提示词示例。

2. **指令微调：** 使用增强后的数据集对预训练语言模型进行指令微调，目标是最大化生成响应和推理依据的似然概率：`log P_Drafter(A, E | Q, D)`。  通过指令微调，Drafter 学会了在给定问题和相关文档的情况下生成合理的答案和推理依据。
``` 

# 有用的知识

  - 如何对召回的文档进行采样
