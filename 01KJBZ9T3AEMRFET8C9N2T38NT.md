---
title: 20240611 - 阅读论文: ACTIVERAG: Revealing the Treasures of Knowledge via Active Learning
confluence_page_id: 2949640
created_at: 2024-06-11T12:05:46+00:00
updated_at: 2024-06-11T12:14:57+00:00
---

# 主要思路

  - 使用四种代理, 对同样的材料进行四个角度的分析
  - 使用 认知连接(Cognitive Nexus), 将四种代理的结果, 放在CoT的结果中, 生成最终结果

文章的重点, 是四种代理的分布, 逐一介绍: 

是的，四个不同的代理会接收到相同的检索段落作为输入，但它们会从各自独特的视角出发，对这些段落进行不同的处理和分析。

以下详细说明每个代理如何处理输入段落：

**1. 语义关联代理 (Associate)**

* **目标:** 将新信息与 LLM 已有的知识网络建立联系，帮助 LLM 更全面地理解问题。  
* **处理方式:**   
* 分析输入段落中的关键词和关键短语。  
* 在 LLM 的知识图谱或其他知识表示中搜索与这些关键词相关的概念和实体。  
* 识别输入段落与 LLM 现有知识之间的语义联系，例如上下位关系、同义关系、因果关系等。  
* **输出:** 与输入段落相关的现有知识，以及这些知识与输入段落之间的语义联系。

**2. 认知锚定代理 (Anchoring)**

* **目标:** 帮助 LLM 理解和学习未知的概念和信息。  
* **处理方式:**   
* 识别输入段落中 LLM 可能不熟悉的概念、实体、事件等。  
* 从知识库中检索关于这些未知信息的定义、解释、例子等。  
* 将这些解释和例子与输入段落中的上下文信息结合起来，帮助 LLM 建立对新知识的理解。  
* **输出:** 对输入段落中未知概念和信息的解释和说明。

**3. 逻辑推理代理 (Logician)**

* **目标:** 利用结构化信息进行逻辑推理，增强 LLM 的问题解决能力。  
* **处理方式:**   
* 从输入段落中提取结构化信息，例如事实、规则、逻辑关系等。  
* 利用逻辑推理引擎或规则引擎，基于提取的结构化信息进行推理。  
* 推断出输入段落中没有直接说明的隐含信息。  
* **输出:** 基于输入段落进行逻辑推理得到的结论和推论。

**4. 认知校准代理 (Cognition)**

* **目标:** 识别并解决新信息与 LLM 现有知识之间可能存在的矛盾，提高答案的准确性。  
* **处理方式:**   
* 对比输入段落与 LLM 现有知识，判断是否存在矛盾或冲突的信息。  
* 如果发现矛盾，评估不同信息的可靠性和可信度。  
* 选择更可靠的信息，或者对矛盾信息进行调和。  
* **输出:** 经过校准和筛选后的信息，以及对可能存在的矛盾的解释。

总而言之，四个代理协同工作，从不同角度分析相同的输入段落，并将各自的分析结果整合到 LLM 的思维链中，帮助 LLM 更全面、准确地理解问题，并生成更可靠的答案。

语义关联代理的提示词: 

You are a cognitive scientist, to answer the following question: 

Question {question}  
I will provide you with several retrieved passages:  
Passages: {passages}  
Task Description:  
Please extract foundational knowledge that may be familiar to the model or advanced information beyond the model’s already familiar foundational knowledge from these passages, and analyze the role of these contents. Summarize and consolidate these contents, which should deepen the model’s understanding of the question through familiarity with these basic and advanced pieces of information. This process aims to encourage the model to comprehend the question more thoroughly and expand its knowledge boundaries. 

  

\---

你是一位认知科学家，需要回答以下问题：

问题：{question}

我将提供一些检索到的段落：

段落：{passages}

任务描述：

请从这些段落中提取模型可能已经熟悉的基础知识，以及超出模型已知基础知识的进阶信息，并分析这些内容的作用。总结并整合这些内容，通过熟悉这些基础和进阶的信息，加深模型对问题的理解。这个过程旨在鼓励模型更全面地理解问题，并扩展其知识边界。

  

  

认知锚定代理的提示词: 

You are a cognitive scientist, to answer the following question: 

Question {question}  
I will provide you with several retrieved passages:  
Passages: {passages}  
Task Description:  
Please extract content that may be unfamiliar to the model from these passages,  
which can provide the model with relevant background and unknown knowledge and concepts, helping it better understand the question. and analyze the role of these contents. 

\---

你是一位认知科学家，需要回答以下问题：

问题：{question}

我将提供一些检索到的段落：

段落：{passages}

任务描述：

请从这些段落中提取模型可能不熟悉的内容，这些内容可以为模型提供相关的背景知识、未知的知识和概念，帮助模型更好地理解问题，并分析这些内容的作用。

逻辑推理代理的提示词: 

You are a logician, to answer the following question:  
Question {question}  
I will provide you with several retrieved passages:  
Passages: {passages}  
Task Description:  
Please extract content from these passages that can help enhance the model’s causal reasoning and logical inference abilities. consolidate these contents, and analyze how the selected information may impact the improvement of the model’s causal reasoning and logical inference capabilities.

\---

你是一位逻辑学家，需要回答以下问题：

问题：{question}

我将提供一些检索到的段落：

段落：{passages}

任务描述：

请从这些段落中提取可以帮助增强模型因果推理和逻辑推理能力的内容，整合这些内容，并分析所选信息如何影响模型因果推理和逻辑推理能力的提升。

  

认知校准代理的提示词: 

Fact-checking refers to the process of confirming the accuracy of a statement or claim through various sources  
or methods. This process ensures that statements or claims are based on reliable and verifiable information while eliminating inaccurate or misleading content. Fact-checking may involve examining data, literature, expert opinions, or other trustworthy sources. In the context of artificial intelligence, model illusion refers to the overconfidence response of the AI. When a model exhibits an ’illusion’ (a tendency to output deceptive data), it indicates that the training data used by the model does not necessarily support the rationality of its outputs. You are a scientist researching fact-checking and addressing model illusions in artificial intelligence.  
To answer the following question:  
Question {question}  
I will provide you with several retrieved passages:  
Passages: {passages}  
Task Description:  
Please extract content from these passages that may be contradictory to the model’s existing knowledge. Identify information that, when added, could update the model’s knowledge and prevent factual errors, alleviating model illusions. Note that these passages are retrieved from the most authoritative knowledge repositories, so they are assumed to be correct.

\---

事实核查是指通过各种来源或方法确认陈述或声明准确性的过程。这个过程确保陈述或声明基于可靠且可验证的信息，同时消除不准确或误导性的内容。事实核查可能涉及检查数据、文献、专家意见或其他可靠来源。在人工智能的背景下，模型幻觉指的是人工智能过于自信的反应。当一个模型表现出“幻觉”（输出欺骗性数据的倾向）时，表明该模型使用的训练数据不一定支持其输出的合理性。你是一位研究人工智能事实核查和解决模型幻觉问题的科学家。

为了回答以下问题：

问题：{question}

我将提供一些检索到的段落：

段落：{passages}

任务描述：

请从这些段落中提取可能与模型现有知识相矛盾的内容。识别添加后可以更新模型知识并防止事实错误的信息，从而减轻模型幻觉。请注意，这些段落是从最权威的知识库中检索到的，因此假定它们是正确的。

  

文章附录中有完整案例
