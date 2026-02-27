---
title: 20250506 - 阅读论文*: PROMPTCOT: Synthesizing Olympiad-level Problems for Mathematical Reasoning in Large Language Models
confluence_page_id: 3801878
created_at: 2025-05-06T12:23:22+00:00
updated_at: 2025-05-06T12:29:38+00:00
---

# 主要思路

  1. 让模型给出"如何构造数据"的步骤 (输入简单的"概念", 要求根据这些概念输出"构造数据的步骤")
  2. 根据构造数据的步骤, 让模型构造数据
  3. 通过"拒绝采样", 选取高质量的数据

更详细的描述: 

您的理解基本正确，但步骤 1 和 2 的描述可以更精确一些。PROMPTCOT 构造数据的核心步骤不仅仅是根据概念输出构造步骤，而是**先生成 rationale，再根据 rationale 和概念生成问题，最终构成（概念，rationale，问题）三元组** 。

更详细的步骤如下：

**1\. 确定概念并收集种子问题：**

首先，确定目标问题的知识领域，并选择相关的数学概念。例如，我们想生成关于几何的问题，可以选择“三角形”、“圆”、“面积”等概念。 然后，从 AoPS 等平台收集一些已有的奥林匹克级别问题作为种子问题，这些问题需要与选择的概念相关。

**2\. 从种子问题中提取概念：**

使用 LLM (例如 Llama-3.1-70B-Instruct) 从种子问题中提取更细化的概念。 这是为了确保 rationale 的生成更贴近种子问题的本质。 这一步的输入是种子问题，输出是一系列概念。 提示词可以参考论文附录 A 中的内容。

**3\. 为种子问题生成 Rationale：**

使用 LLM 根据提取的概念和目标难度（例如奥林匹克级别）为每个种子问题生成 rationale。 这一步的输入是概念和种子问题，输出是 rationale。 提示词可以参考论文附录 B 中的内容。 这里生成的 rationale 解释了如何从给定的概念出发，一步步构造出种子问题。

**4\. 构造数据：**

将步骤 2 提取的概念、步骤 3 生成的 rationale 和原始的种子问题组成一个三元组（概念，rationale，问题）。 收集大量的这样的三元组就构成了训练数据集。

**5\. 拒绝采样：**

使用两个独立的 LLM 评估器对生成的数据进行评估，只有获得两个评估器一致认可的数据才会被保留。 评估器的提示词可以参考论文附录 C 中的内容。 这一步的目的是筛选高质量的数据，用于训练问题生成模型。

**6\. 训练问题生成模型：**

使用筛选后的高质量数据训练问题生成模型。 这个模型也是一个 LLM，例如 Llama-3.1-8B。 模型学习的目标是根据给定的概念和 rationale 生成相应的问题。

**7\. 生成新问题：**

训练完成后，可以使用这个模型生成新的问题。 输入一组概念，模型会生成相应的 rationale 和问题。

**关键区别：**

您之前的理解中，步骤 1 和 2 是根据概念直接构造数据。 而 PROMPTCOT 的做法是先从种子问题中提取概念，然后为种子问题生成 rationale，最终用（概念，rationale，问题）三元组来训练模型。 rationale 的引入是 PROMPTCOT 的核心，它使得模型能够学习问题设计的内在逻辑，从而生成更具挑战性的问题。

# 详细描述

让我们通过一个例子详细说明 PROMPTCOT 的步骤。

**目标：**生成一道关于排列组合的奥林匹克级别数学题。

**1\. 概念提取:**

首先，我们确定几个相关的数学概念：

  - 排列
  - 组合
  - 容斥原理

**2\. Rationale 生成:**

接下来，我们使用一个大型语言模型 (LLM)，例如 Llama-3.1-70B-Instruct，根据上面提取的概念和目标难度，生成 rationale。 提示词可以参考论文附录 B 中的内容。

  - **输入给 LLM 的提示词 (示例):**

    
    
    Imagine you are an expert in educational problem design.
    You will be shown these components:
    Fundamental Concepts: 排列, 组合, 容斥原理
    Difficulty Level: 奥林匹克级别
    Your task is to reverse-engineer a clear thinking process that shows how a teacher might design this problem.  This thinking process should:
    • Show how combining the given foundational concepts naturally leads to a problem at the specified difficulty level.
    • Include all key decisions and reasoning that shaped the problem design.
    • (IMPORTANT) Be so precise and detailed that another teacher following these exact steps would recreate the identical problem.
    • (IMPORTANT) Be so natural and logical that another teacher could derive the same thinking process using only the foundational concepts and difficulty level.
    Present your answer after “Thinking Process:” with the complete step-by-step thinking process described above.
    

**  
**

  - **LLM 生成的 Rationale (示例):**

    
    
    Thinking Process:
    1. 我们想要设计一道关于排列组合的奥林匹克级别数学题。
    2. 为了体现奥林匹克级别的难度，我们需要结合多个概念，并且引入一些限制条件。
    3. 我们可以考虑一个计数问题，例如计算满足特定条件的排列或组合的数量。
    4. 为了结合容斥原理，我们可以设定一些互斥的条件，并要求计算满足至少一个条件的排列或组合的数量。
    5. 具体来说，我们可以考虑一个由 n 个不同元素组成的集合，并要求计算其中至少包含 k 个特定元素的子集的数量。
    6. 为了进一步增加难度，我们可以引入一些额外的限制条件，例如子集的大小必须是偶数。
    

**  
**

**3\. 问题生成模型训练:**

现在我们有了许多类似的（概念，rationale，问题）三元组。 我们可以用这些数据来训练一个问题生成模型。 这个模型也是一个 LLM，例如 Llama-3.1-8B。

  - **训练数据 (示例):**

    
    
    (概念: 排列, 组合, 容斥原理; Rationale:  [如上]; 问题: 从 1 到 10 的整数中选取一个子集，要求子集的大小是偶数，并且至少包含 1, 2, 3 中的两个数。求这样的子集有多少个。)
    

模型学习的目标是根据给定的概念和 rationale 生成相应的问题。

**4\. 拒绝采样:**

为了保证生成问题的质量，PROMPTCOT 使用了拒绝采样。 两个独立的 LLM 评估器会对生成的问题进行评估，只有获得两个评估器一致认可的问题才会被保留用于进一步训练模型。 评估器的提示词可以参考论文附录 C 中的内容。

**5\. 推理:**

训练完成后，我们可以使用这个模型来生成新的问题。 我们只需要提供一组概念，模型就会生成相应的 rationale 和问题。

**总结:**

通过以上步骤，PROMPTCOT 可以生成高质量的奥林匹克级别数学题。 rationale 的作用在于指导问题生成模型学习如何设计问题，使其能够生成更具挑战性和创造性的题目。 拒绝采样机制则保证了生成问题的质量。
