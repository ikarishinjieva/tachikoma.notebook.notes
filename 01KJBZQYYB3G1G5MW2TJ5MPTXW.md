---
title: 20250131 - 阅读论文: CPO: Chain of Preference Optimization: Improving Chain-of-Thought Reasoning in LLMs
confluence_page_id: 3344045
created_at: 2025-01-30T16:40:10+00:00
updated_at: 2025-01-30T16:40:10+00:00
---

# 主要思路

CPO方法主要是通过DPO训练, 让模型在CoT选取下一步骤时 (根据问题和之前的步骤) , 习得如何选择更好的步骤.

# 其他重要信息

你的总结抓住了 CPO 的核心目标，即通过 DPO 训练提升 CoT 推理中每一步的选择质量。但是，它遗漏了一些重要的细节，使得描述不够完整和准确。

以下是一些你应该补充的关键信息：

  - **CPO 如何获得训练数据：** 你的总结没有提到 CPO 的训练数据来源。CPO 的训练数据并非凭空而来，而是利用 ToT (Tree of Thought) 搜索过程生成的。ToT 会探索多种推理路径，并对每一步的多个候选思维进行评估。CPO 正是利用 ToT 产生的这些数据来学习 ToT 的偏好。

  - **偏好信息的概念：** 你的总结没有明确提到“偏好信息”的概念。CPO 的核心在于学习 ToT 的“偏好”，即 ToT 认为哪些思维步骤更有利于最终解决问题。CPO 通过学习这些偏好信息，才能在 CoT 推理过程中选择更好的步骤。

  - **链级优化：** 你的总结只提到了“选择更好的步骤”，但没有强调 CPO 是在链级进行优化的。CPO 会针对推理链中的每一步都进行优化，而不是仅仅关注最终答案。 这与 FPO (Full-path Preference Optimization) 的区别非常重要，因为 FPO 容易出现 LCP 梯度消失问题，而 CPO 可以避免这个问题。

一个更完整的总结可以是这样的：

CPO (Chain of Preference Optimization) 方法的核心思想是利用 ToT (Tree of Thoughts) 搜索过程中产生的偏好信息，通过 DPO (Direct Preference Optimization) 算法训练 LLM，使其在 CoT (Chain of Thoughts) 推理过程中，学习如何选择更有利于解决问题的推理步骤。CPO 通过对 ToT 搜索树中每一步的多个候选思维进行比较，提取出 ToT 的偏好信息，并以此作为训练数据。 与仅仅关注最终答案的 FPO (Full-path Preference Optimization) 不同，CPO 在链级进行优化，从而避免了 LCP 梯度消失问题，更有效地提升了 LLM 的推理能力。

改进后的总结更清晰地表达了 CPO 方法的核心思想和关键细节，使其更易于理解和与其他方法进行区分。
