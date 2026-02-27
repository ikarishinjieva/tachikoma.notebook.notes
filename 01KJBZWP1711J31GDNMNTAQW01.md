---
title: 20250807 - 阅读论文*: Beyond the 80/20 Rule: High-Entropy Minority Tokens Drive Effective Reinforcement Learning for LLM Rea
confluence_page_id: 4161902
created_at: 2025-08-07T08:55:05+00:00
updated_at: 2025-08-07T08:55:05+00:00
---

# 主要思路

作者首先分析了LLM在进行思维链（Chain-of-Thought, CoT）推理时生成的文本，发现了两个关键的**熵模式（Entropy Patterns）** ：

  - **模式一：熵分布的“二八定律”** 。绝大多数（约80%）的词元是以极低的熵生成的，它们非常确定；只有一小部分（约20%）的词元是以高熵生成的，充满了不确定性。
  - **模式二：高/低熵词元的功能差异** 。
    - **低熵词元** ：通常是用来完成当前句子结构或单词拼写的，如单词后缀、代码片段等，起着“**沿路执行** ”的作用。
    - **高熵词元** ：通常是逻辑连接词、假设词或表示转折的词（如 "however", "thus", "assume"），它们在推理路径中起到了**“分叉路口”（forks）**的作用，决定了后续推理的方向。

为了验证“分叉路口”这一假设，作者通过实验**量化地证明** ：在推理时，适度提高这些高熵“分叉词元”的生成随机性（温度/熵），可以提升模型的推理性能；反之，降低其随机性则会导致性能下降。

对应的方法: ms-swift提供了 --top_entropy_quantile 控制训练范围, 使用 --log_entropy 观察熵值的变化

# 相关论文

讨论熵对RL过程的改进: [20250807 - 阅读论文*: The Entropy Mechanism of Reinforcement Learning for Reasoning Language Models]
