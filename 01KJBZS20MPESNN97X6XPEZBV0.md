---
title: 20250318 - 阅读论文*: Chain of Draft: Thinking Faster by Writing Less
confluence_page_id: 3801197
created_at: 2025-03-18T06:57:26+00:00
updated_at: 2025-03-18T12:43:01+00:00
---

# 主要思想

让CoT只输出关键词句, 而不用输出完善信息 (每一步输出5个单词). 这样即控制了思考方向, 又不限制模型的思考发散. 

举例: 

```
我们用一个更复杂的例子来对比 CoT 和 CoD，并展示 CoD 的提示词。

问题：

小明有 10 个苹果，小红有 6 个苹果。小明给小红 2 个苹果后，小明比小红多几个苹果？

CoT 提示词：

Think step by step to answer the following question. Return the answer at the end of the response after a separator ####.

Q: 小明有 10 个苹果，小红有 6 个苹果。小明给小红 2 个苹果后，小明比小红多几个苹果？

CoT 可能的输出：

小明一开始有 10 个苹果。
小红一开始有 6 个苹果。
小明给了小红 2 个苹果，所以小明现在有 10 - 2 = 8 个苹果。
小红得到了 2 个苹果，所以小红现在有 6 + 2 = 8 个苹果。
小明比小红多 8 - 8 = 0 个苹果。

####
0

CoD 提示词：

Think step by step, but only keep a minimum draft for each thinking step, with 5 words at most. Return the answer at the end of the response after a separator ####.

Q: 小明有 10 个苹果，小红有 6 个苹果。小明给小红 2 个苹果后，小明比小红多几个苹果？

CoD 可能的输出：

小明 10 - 2 = 8
小红 6 + 2 = 8
8 - 8 = 0

####
0

对比分析：

可以看到，CoT 的输出详细地解释了每一步的计算过程和原因，而 CoD 则只保留了最核心的算式。CoD 的输出更加简洁，减少了冗余信息，从而降低了 token 消耗和延迟。

CoD 提示词的关键:

"Think step by step, but only keep a minimum draft for each thinking step"  明确指示模型进行逐步推理，但只保留最关键的信息。

"with 5 words at most"  鼓励模型尽可能简洁地表达每个步骤，但这只是一个建议，并非强制限制。

"Return the answer at the end of the response after a separator ####"  指示模型在最终答案前使用分隔符，方便提取答案。

这个例子更清晰地展示了 CoD 如何通过简洁的推理步骤来提高效率，同时保留核心推理过程。
```
