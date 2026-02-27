---
title: 20250227 - 阅读论文*: 迈向复现 OpenAI o1 的一小步：Steiner 开源模型解析
confluence_page_id: 3344425
created_at: 2025-02-27T12:58:39+00:00
updated_at: 2025-02-27T12:58:39+00:00
---

# URL

<https://mp.weixin.qq.com/s/9v3Y776ie_4j9zfSr0Mlsg>

# 有用的思路

  1. 作者通过 reasoning tokens 的数量线性增长关系, 推理出o1对于决策树的遍历是线性方法, 而不是树形并发方法
     1. 这种线性遍历听起来很浪费，但与并行搜索相比，它具有三个极其优秀的性质：

        - • 1.前置的尝试无论对错都在 context memory 中，每一步决策均基于过去的完整信息；

        - • 2.隐含的 backtracking 无需保证目标节点已经在搜索树内，探索更加自由；

        - • 3.工程层面可以复用一切现有的经过高度优化的推理 infrastructure。

  2. 如何构造带backtracking的COT流程 (使用LLM生成的COT流程不带backtracking)
     1. 我设计了两种数据合成与扩增方法：

        - • 1.对上述 shortcut 数据集进行随机截断，并隐藏正确答案，让强大的 LLM 基于被截断后的前缀去尝试正向推理一定的 steps，然后再提供正确答案，以获得 backtracking 样本；

        - • 2.对上一步产生的 steps 进行聚类后赋予唯一 ID，将同一个问题下的所有 steps 构建为有向无环图 (DAG)，并对 DAG 进行随机采样以获得多项式量级的 reasoning path 样本。

  3. RL阶段的reward设计, 是基于DAG的一套权重系统
     1. 3.Reinforcement Learning with Step-Level Reward (RL)：经过前两阶段，模型已经学会生成和补完 reasoning path，但它还不知道哪些是正确且高效的选择。如果我们盲目 reward 更短的 reasoning path，则会降级为 shortcut 学习来 internalize CoT。这里我设计了一种启发式的 reward 机制：基于 DAG 中每个节点的入边数量 e_i、出边数量 e_o、距离原始问题的距离 d_s、距离正确答案的距离 d_e 来加权，为每个 step 以及整个 reasoning path 赋予 reward，以引导模型学会平衡探索的广度和深度。
