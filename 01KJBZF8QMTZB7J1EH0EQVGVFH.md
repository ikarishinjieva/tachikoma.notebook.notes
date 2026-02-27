---
title: 20240907 - 阅读论文*: STRUCTURED-RAG: JSON RESPONSE FORMATTING WITH LARGE LANGUAGE MODELS
confluence_page_id: 3342356
created_at: 2024-09-07T16:05:15+00:00
updated_at: 2024-09-07T16:05:28+00:00
---

# 论文目标

讨论 "让LLM生成Json格式" 的相关方法, 重点是以下几种方法: 

  1. StructuredRAG基准测试的六项任务：

     - 这是论文的核心内容，提供了评估框架的基础。
     - 大量篇幅用于描述这些任务及其设计理念。
  2. 两种提示策略：f-String提示和Follow the Format (FF)提示：

     - 这是论文的主要研究方法之一。
     - 详细比较了这两种策略在不同任务和模型上的表现。
  3. 多模型比较：

     - 使用多个模型（如Gemini 1.5 Pro、Llama 3 8B-instruct、GPT-3.5等）进行测试。
     - 提供了不同模型在结构化输出任务上的性能对比。
  4. OPRO优化方法：

     - 作为一种改进技术被引入，特别是针对某些特定任务。
     - 虽然不是主要研究对象，但在解决特定问题时显示了重要性。

# StructuredRAG基准测试的六项任务

评估LLM的逻辑能力, 顺便评估JSON结构化输出的能力

```
StructuredRAG基准测试包含六项任务，旨在评估大语言模型生成结构化输出的能力。以下是这六项任务的详细描述：

1. AnswerQuestion
   - 输出类型：string
   - 任务：根据给定的上下文回答问题
   - 示例输出：{"answer": "The Knesset"}

2. ContextScore
   - 输出类型：integer
   - 任务：评估给定上下文与问题的相关性，打分范围为0-5
   - 示例输出：{"context_score": 4}

3. IsAnswerable
   - 输出类型：boolean
   - 任务：判断给定的问题是否可以根据提供的上下文回答
   - 示例输出：{"is_answerable": true}

4. ParaphraseQuestions
   - 输出类型：List[string]
   - 任务：生成给定问题的3个改写版本
   - 示例输出：{"paraphrased_questions": ["What is Israel's parliament called?", "Can you name the legislative body of Israel?", "What is the name of the Israeli legislative assembly?"]}

5. AnswerWithConfidence
   - 输出类型：复合对象（string + integer）
   - 任务：回答问题并提供置信度评分（0-5）
   - 示例输出：{"answer": "The Knesset", "confidence": 5}

6. GenerateAnswersWithConfidences
   - 输出类型：List[AnswerWithConfidence]
   - 任务：生成多个可能的答案，每个答案都带有置信度评分
   - 示例输出：{"answers": [{"answer": "The Knesset", "confidence": 5}, {"answer": "Israeli Parliament", "confidence": 4}]}

这些任务设计涵盖了不同的输出类型和复杂度：
- 简单类型：字符串、整数、布尔值
- 复杂类型：字符串列表、复合对象、复合对象列表

这种设计允许全面评估语言模型在不同难度和复杂度的结构化输出任务中的表现。任务的内容与检索增强生成（RAG）系统相关，涉及问题回答、上下文评估、问题改写等常见操作，使得这个基准测试对实际应用场景具有较强的相关性。
``` 

# 两种提示策略：f-String提示和Follow the Format (FF)提示

![image2024-9-7 23:54:22.png](/assets/01KJBZF8QMTZB7J1EH0EQVGVFH/image2024-9-7%2023%3A54%3A22.png)

FF的评估结论会好很多, FF的例子: 

````
当然，我很乐意为您提供一个例子来说明 Follow the Format (FF) 提示和 f-string 提示的不同。让我们以 "GenerateAnswersWithConfidences" 任务为例。

假设我们要让模型回答一个问题，并提供答案的置信度。

1. f-string 提示示例：

```
问题：巴黎是哪个国家的首都？

请提供答案和置信度，格式如下：
{
    "answer": "你的答案",
    "confidence": 0.0到1.0之间的数字
}
```

2. Follow the Format (FF) 提示示例：

```
问题：巴黎是哪个国家的首都？

请按照以下格式提供答案和置信度：

{
    "answer": "示例答案",
    "confidence": 0.95
}

现在，请使用上述格式回答问题：巴黎是哪个国家的首都？
```

主要区别：

1. 具体示例：
   - FF提示提供了一个具体的填充示例，而f-string只给出了结构。
   - FF中的 "示例答案" 和 "0.95" 给出了更明确的指导。

2. 格式强调：
   - FF提示更强调遵循给定的格式，通过展示完整的JSON结构。
   - f-string提示可能更依赖于模型对JSON格式的先验理解。

3. 两步指令：
   - FF提示通常包含两个步骤：首先展示格式，然后要求使用该格式回答。
   - f-string提示往往将格式要求和问题回答合并在一起。

4. 细节程度：
   - FF提示通常提供更多细节，如示例中的具体置信度值。
   - f-string提示可能只给出值的范围或类型。

5. 模仿效果：
   - FF提示鼓励模型更直接地模仿给定的格式。
   - f-string提示可能给模型更多解释空间，但也可能导致更多变化。

6. 长度和复杂性：
   - FF提示通常更长，包含更多信息。
   - f-string提示相对简洁，但可能不够明确。

这种区别使得 FF 提示在帮助模型生成正确的结构化输出方面更有效，特别是对于可能不太熟悉特定格式要求的模型。然而，FF 提示也需要更长的输入，这在某些应用场景中可能是一个考虑因素。
```` 

# 多模型比较

简述: 

  1. Gemini 1.5 Pro 总体表现最好

  2. FF提示策略更好

```
论文中对不同模型在结构化输出任务上的性能进行了对比。以下是主要的对比结果：

1. 模型整体表现：
   - Gemini 1.5 Pro 总体表现最好，在大多数任务中都显示出卓越的性能。
   - GPT-3.5和GPT-4也表现出色，特别是在使用 Follow the Format (FF) 提示策略时。
   - Llama 系列模型（如 Llama 3 8B-instruct）表现相对较弱，但仍然能够完成任务。

2. 提示策略影响：
   - 所有模型在使用 FF 提示策略时性能都有所提升。
   - 较小的模型（如 Llama 系列）从 FF 策略中获得的相对收益更大。

3. 任务特定表现：
   - 在复杂任务（如 GenerateStructuredOutput）中，较大模型（Gemini 1.5 Pro, GPT-4）的优势更明显。
   - 在简单任务（如 ClassifyContent）中，模型间的差距较小。

4. JSON 格式正确性：
   - 大型模型（Gemini 1.5 Pro, GPT-4）在生成正确 JSON 格式方面表现出色。
   - 较小模型在使用 FF 策略后，JSON 格式正确性显著提高。

5. 内容准确性：
   - 大型模型在内容准确性方面普遍优于小型模型。
   - FF 策略帮助提高了所有模型的内容准确性，但提升幅度因模型而异。

6. 置信度评估：
   - 在 GenerateAnswersWithConfidences 任务中，大型模型表现出更准确的自我评估能力。
   - 较小模型的置信度评估可能不太可靠，但使用 FF 策略后有所改善。

7. 跨任务一致性：
   - 大型模型（如 Gemini 1.5 Pro）在不同任务间表现出更高的一致性。
   - 较小模型的表现在不同任务间波动较大。

8. 错误类型：
   - 大型模型的错误主要集中在内容方面，很少有格式错误。
   - 较小模型使用 f-string 策略时可能出现更多格式错误，但使用 FF 策略后这种情况大幅改善。

9. 模型规模效应：
   - 总体而言，模型规模越大，在结构化输出任务上的表现越好。
   - 但是，即使是较小的模型，在使用适当的提示策略（如 FF）后，也能显著提升性能。

10. 泛化能力：
    - 大型模型表现出更强的泛化能力，能够更好地处理未见过的任务结构。
    - 较小模型在遇到新的任务结构时可能表现不稳定，但 FF 策略有助于改善这一点。

总结来说，研究结果表明，虽然大型模型（如 Gemini 1.5 Pro 和 GPT-4）在结构化输出任务上表现最佳，但通过使用适当的提示策略（特别是 Follow the Format），即使是较小的模型也能显著提高其性能。这种性能提升在格式正确性和内容准确性两个方面都有体现，但提升的程度因模型和具体任务而异。这些发现为如何更有效地利用不同规模的 LLMs 进行结构化输出任务提供了重要的指导。
``` 

# OPRO优化方法

没有步骤描述
