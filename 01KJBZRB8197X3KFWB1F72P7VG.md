---
title: 20250207 - 阅读论文*: From RAG to RICHES: Retrieval Interlaced with Sequence Generation
confluence_page_id: 3344207
created_at: 2025-02-07T12:34:22+00:00
updated_at: 2025-02-07T12:34:22+00:00
---

# 主要思路

  1. 在LLM思考的过程中, 规定单独的步骤, 要求LLM生成检索词
  2. 生成检索词的过程中, 在解码(进行argmax)时, 要求检索词(的前缀)必须出现在语料库中 (否则将其概率置为0, 这样只能生成在语料库中出现的词)
  3. 将语料库中 检索词出现的句子, 放在LLM的答案中
  4. 让LLM继续思考

举例: (白色部分是检索词, 绿色部分是检索到的结果)

![image2025-2-7 20:27:31.png](/assets/01KJBZRB8197X3KFWB1F72P7VG/image2025-2-7%2020%3A27%3A31.png)

方案的优点:

  - 检索词的前缀是大模型"想说的话", 只是被限制在语料库中. 与RAG的搜索过程不同, 检索词的生成方向是大模型自己的意志. 
  - 这样产生的搜索方向, 一定程度上是带逻辑的

方案的缺点: 

  - 检索词生成的方向, 还是受到 语料库中的素材的影响, 可能无法产生正确的逻辑方向. (或者说: 通过"前缀匹配"来限制大模型的思考, 不一定是有效的限制思考的方式, 限制思考的主要目的是让模型集中精力, 而不是损伤逻辑)

# 其他信息

RICHES 的核心修改确实在于将原本 decoder 中的 argmax 步骤（或者更广义地说，是 token 选择的步骤）修改为需要检索 FM 索引。 但这仍然不仅仅是简单的修改一行代码，而需要考虑以下几个方面：

  1. **与 FM 索引的交互:** 需要实现一个高效的接口，使得 decoder 能够在每一步生成过程中与 FM 索引进行交互。 这包括将生成的 token 前缀传递给 FM 索引，并获取可能的后续 token 列表以及它们对应的概率或分数。

  2. **概率分布的调整:** FM 索引返回的后续 token 列表及其分数需要与 LLM 自身的概率分布进行整合。 这可能涉及到将两者的分数进行加权平均，或者使用其他更复杂的融合策略。

  3. **缓存机制:** 由于每次生成 token 都需要与 FM 索引进行交互，为了提高效率，需要实现一个缓存机制，将已经查询过的前缀及其对应的结果缓存起来，避免重复计算。

  4. **回溯机制:** 在生成过程中，如果遇到无法在 FM 索引中找到匹配的后续 token 的情况，需要实现一个回溯机制，尝试其他的生成路径。

  5. **参数调整:** 需要对一些参数进行调整，例如 FM 索引的查询参数、概率融合的权重等，以获得最佳性能。

**代码示例 (概念性):**
    
    
    # 原始的 argmax 解码
    next_token = torch.argmax(logits, dim=-1)
    
    # 修改后的 RICHES 解码 (概念性代码，并非完整实现)
    prefix = generated_tokens.tolist()
    candidates, scores = query_fm_index(prefix)  # 查询 FM 索引
    lm_scores = logits[0, -1, candidates] # 获取 LLM 对候选词的概率
    combined_scores = combine_scores(lm_scores, scores) # 融合 LLM 和 FM 索引的分数
    next_token = candidates[torch.argmax(combined_scores)] # 选择分数最高的 token
    

**总结:**

虽然 RICHES 的核心修改在于 token 选择的步骤，但这仍然需要对 decoder 进行 substantial 的修改，包括与 FM 索引的交互、概率分布的调整、缓存机制、回溯机制以及参数调整等。 这比简单的修改 argmax 步骤要复杂得多。
