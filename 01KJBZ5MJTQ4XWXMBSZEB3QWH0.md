---
title: 20240205 - 阅读论文: SELF-RAG: LEARNING TO RETRIEVE, GENERATE, AND CRITIQUE THROUGH SELF-REFLECTION
confluence_page_id: 2590032
created_at: 2024-02-05T05:46:50+00:00
updated_at: 2024-02-06T09:00:31+00:00
---

SELF-RAG: LEARNING TO RETRIEVE, GENERATE, AND CRITIQUE THROUGH SELF-REFLECTION

<https://app.txyz.ai/chat/36012977-cf44-486f-8216-f1f766684e0f>

# 目录

# 问答

```
用中文回答: 这篇论文主要解决什么问题
这篇论文主要解决的问题是针对大型语言模型的属性问题回答（Attributed Question Answering）进行评估和建模。该论文提出了一种针对属性问题回答的评估方法，并通过对大型语言模型进行建模来解决这个问题。

---

什么是属性问题回答
属性问题回答（Attributed Question Answering）是指在问答任务中，针对问题中的属性进行回答的问题类型。这种问题通常需要从给定的文本或知识库中找到与属性相关的信息，并生成相应的回答。属性问题回答是自然语言处理和人工智能领域的一个重要研究方向，旨在提高问答系统对于属性类问题的准确性和效果。

---

举一个属性问题回答的例子
一个属性问题回答的例子是：给定问题 "Who is the author of the paper 'Attributed Question Answering: Evaluation and Modeling for Attributed Large Language Models'?"，可以通过搜索相关的文献或知识库来找到与该属性相关的信息，并生成回答 "The authors of the paper 'Attributed Question Answering: Evaluation and Modeling for Attributed Large Language Models' are Bernd Bohnet, Vinh Q Tran, Pat Verga, Roee Aharoni, Daniel Andor, Livio Baldini Soares, Jacob Eisenstein, Kuzman Ganchev, Jonathan Herzig, Kai Hui, et al."

---

为什么要单独讨论属性问题回答
属性问题回答是指回答关于某个实体或事物的特定属性的问题，例如作者、标题、发布日期等。与其他类型的问题不同，属性问题回答通常需要从文献、知识库或其他信息源中查找相关信息，并提取出所需的属性值。由于属性问题回答的特殊性，需要单独讨论其解决方法和技术。

---

本文的逻辑主线是什么
本文的逻辑主线是介绍了一种名为Self-Reflective Retrieval-Augmented Generation (SELF-RAG)的新框架，该框架通过检索和自我反思来提高语言模型的质量和准确性。文章首先指出了大型语言模型（LLMs）在生成回答时常常存在事实错误的问题，然后介绍了Retrieval-Augmented Generation (RAG)方法作为一种减少错误的方法。然而，传统的RAG方法存在一些问题，如无差别地检索和整合一定数量的检索段落，无论检索是否必要或段落是否相关，这可能降低了语言模型的灵活性或导致无用的回答生成。为了解决这些问题，作者提出了SELF-RAG框架，该框架训练一个单一的任意语言模型，该模型可以根据需求自适应地检索段落，并使用特殊的反思标记生成和反思检索的段落和自身生成的内容。通过生成反思标记，语言模型在推理阶段具有可控性，可以根据不同的任务要求调整其行为。实验证明，SELF-RAG在多个任务上显著优于现有的LLMs和检索增强模型，特别是在开放领域的问答、推理和事实验证任务中，以及在长篇生成中提高事实准确性和引用准确性方面取得了显著的改进。

---

self-rag的主要步骤是什么
SELF-RAG的主要步骤如下：

1. 检索决策（Retrieve Decision）：根据输入提示和之前生成的内容，模型预测是否需要进行检索。如果需要检索，进入下一步；如果不需要检索，直接生成下一个输出段落。

2. 检索（Retrieval）：如果决定进行检索，模型使用检索器（Retriever）根据输入提示和之前生成的内容来检索相关的文本段落。模型同时生成一个检索标记（retrieval token），用于调用检索器。

3. 生成（Generation）：模型同时处理多个检索到的文本段落，并根据任务要求生成相应的输出段落。

4. 评估（Critique）：模型生成批判标记（critique token），用于评估自己生成的输出段落的质量和相关性。模型会评估检索到的文本段落是否提供了有用的信息，并评估生成的输出段落是否与检索到的段落一致。

5. 输出排序（Ranking）：根据评估结果，模型对生成的输出段落进行排序，选择最佳的输出段落作为最终结果。

以上是SELF-RAG框架的主要步骤，通过自适应地检索和自我反思，该框架可以提高语言模型的质量和准确性。

---

self-rag框架的主要改进在评估阶段么?
SELF-RAG框架的主要改进在评估阶段是通过引入自我反思机制来提高模型生成结果的质量和准确性。在评估阶段，SELF-RAG模型会生成特殊的反思标记（reflection tokens），用于评估自己生成的输出段落的相关性、支持性和完整性。这些反思标记包括ISREL（判断检索到的文本段落是否相关）、ISSUP（判断生成的输出段落是否得到了支持）和ISUSE（评估生成的输出段落是否对输入有用）。通过引入这些反思标记，SELF-RAG可以对生成的输出段落进行评估和排序，选择最佳的输出结果。

相比传统的RAG方法，SELF-RAG在评估阶段引入了更加细粒度的评估机制，可以更好地判断检索到的文本段落是否提供了有用的信息，并评估生成的输出段落是否与检索到的段落一致。这样可以提高模型生成结果的准确性和相关性，从而增强了模型的质量和可靠性。

---

除了评估阶段, self-rag框架还做了哪些改进
除了评估阶段，SELF-RAG框架还做了以下改进：

1. 检索增强：SELF-RAG通过引入检索机制，可以根据输入提示和之前生成的文本来检索相关的文本段落。这样可以提供更多的信息和支持，帮助模型生成更准确和有用的输出。

2. 自我反思：SELF-RAG引入了自我反思机制，通过生成特殊的反思标记（reflection tokens）来评估和批判自己生成的输出。这些反思标记包括ISREL（判断检索到的文本段落是否相关）、ISSUP（判断生成的输出段落是否得到了支持）和ISUSE（评估生成的输出段落是否对输入有用）。通过自我反思，SELF-RAG可以对生成的输出进行评估和排序，选择最佳的输出结果。

3. 硬控制和软约束：SELF-RAG在生成过程中使用自己生成的反思标记来施加硬控制或软约束。这样可以更好地控制生成的输出，使其符合特定的要求和标准。

总的来说，SELF-RAG框架在检索增强、自我反思和生成过程控制方面都进行了改进，以提高模型的质量、准确性和可靠性。
``` 

# 推理过程

<https://github.com/AkariAsai/self-rag>

Self-RAG 训练了两个模型: Critic 和 Generator:

  - Critic 好像是从GPT-4进行蒸馏, 用于标记 反思标记, 为Generator的训练提供训练数据
  - Generator 生成带标记的对话

模型的推理过程如下: 

![image2024-2-6 15:57:44.png](/assets/01KJBZ5MJTQ4XWXMBSZEB3QWH0/image2024-2-6%2015%3A57%3A44.png)

  1. 根据问题x, 和之前的输出y
  2. 如果 Retrieve标记为True (需要查询文档库)
     1. 使用(x, y
     2. 对于每一份文档d, 根据 (x,d) 生成IsRel标记 (证据是否为输入提供了有效信息). 并根据(x,d,y
     3. 根据 (x,d,y) 生成 IsSup (输出的每一部分是否都有证据支撑) 和IsUse标记 (答案和问题的匹配程度)
     4. 根据标记对文档质量进行排序
  3. 如果 Retrieve标记为False (不需要查询文档库)
     1. 根据问题x, 生成输出y
     2. 根据(x, y), 生成IsUse标记 (答案和问题的匹配程度)

# Critic模型的训练过程

## 数据准备

<https://github.com/AkariAsai/self-rag/tree/main/data_creation/critic/gpt4_reward>

### IsUse标记 的训练数据 (答案和问题的匹配程度)

提示词: 给定问题和答案, 将Utility (有效程度, 答案对问题是否有效的) 划分为5级

```
Given an instruction and an output, rate whether the response appears to be a helpful and informative answer to the query, from 1 (lowest) - 5 (highest). We call this score perceived utility.

The detailed criterion is as follows:
5: The response provides a complete, highly detailed, and informative response to the query, fully satisfying the information needs.
4: The response mostly fulfills the need in the query, while there can be some minor improvements such as discussing more detailed information, having better structure of the response, or improving coherence. 
3: The response is acceptable, but some major additions or improvements are needed to satisfy users' needs.
2: The response still addresses the main request, but it is not complete or not relevant to the query.
1: The response is barely on-topic or completely irrelevant.

##
Instruction: Who is the current prime minister of the UK as of 2023?
Output: Boris Johnson was the prime minister of the UK from 2019 - 2022.
Perceived utility: 2
Explanation: While the output provides a factually correct statement about the UK prime minister from 2019 to 2022, this instruction asks who the prime minister is as of 2023, so it doesn't answer the instruction. Therefore, the utility is 2.

##
Instruction: Given a description of a travel destination, recommend 10 tourist attractions with detailed explanations of each. The travel destination is Tokyo, Japan.
Output: 'Tokyo is a vibrant city full of exciting tourist attractions. Some must-see sights include the Tokyo Skytree, Tokyo Disneyland, Sensoji Temple, Meiji Shrine, Tsukiji Fish Market, Harajuku, and Shinjuku Gyoen.
Perceived utility: 3
Explanation: This output doesn't provide descriptions of each attraction and the number of the attractions is also less than 10. While this output partially answers the instructions, it doesn't match the instructions strictly. 

##
Instruction: {instruction}
Output:{output}
``` 

### Retrieve标记 的训练数据 (证据是否能完全证明输出, 是否需要更多证据)

提示词: 给定 输入/证据/输出, 判断证据是否能完全证明输出

  - 如果证据能完全证明输出, [Continue to Use Evidence]
  - 如果输出不需要任何证明, [No Retrieval]
  - 如果需要其他证据才能证明输出, [Retrieval]

```
You will be provided with an instruction, evidence, output sentence, and preceding sentences (optional). If the preceding sentence is given, the output should be the sentence that follows those preceding sentences.  Your task is to determine whether the information in the output sentence can be fully verified by the evidence or if it requires further external verification. There are three cases:
- If the output sentence can be verified solely with the evidence, then respond with [Continue to Use Evidence]. 
- If the sentence doesn't require any factual verification (e.g., a subjective sentence or a sentence about common sense), then respond with  [No Retrieval]. 
If additional information is needed to verify the output sentence, respond with [Retrieval]. Please provide explanations for your judgments. 

##
Instruction: Explain the use of word embeddings in Natural Language Processing.
Preceding sentences: Word embeddings are one of the most powerful tools available for Natural Language Processing (NLP). They are mathematical representations of words or phrases in a vector space, allowing similarities between words and the context in which they are used to be measured. 
Evidence:
Word embedding
Word embedding is the collective name for a set of language modeling and feature learning techniques in natural language processing (NLP) where words or phrases from the vocabulary are mapped to vectors of real numbers. Conceptually it involves a mathematical embedding from a space with one dimension per word to a continuous vector space with a much lower dimension. 
Output: Word embeddings are useful for tasks such as sentiment analysis, text classification, predicting the next word in a sequence, and understanding synonyms and analogies.
Rating: [Retrieval]
Explanation: The output discusses the applications of word embeddings, while the evidence only discusses the definitions of word embeddings and how it works. Therefore, we need to retrieve other evidence to verify whether the output is correct or not.

###
Instruction: {instruction}
Preceding sentences: {preceding_sentences}
Evidence: {evidence}
Output: {target_output}
Rating:
``` 

### IsRel标记的训练 (证据是否为输入提供了有效信息)

提示词: 给定输入指令和证据, 判断证据为指令提供了有效的信息, 可用于完成指令的任务. 

```
You'll be provided with an instruction, along with evidence and possibly some preceding sentences. 
When there are preceding sentences, your focus should be on the sentence that comes after them. 
Your job is to determine if the evidence is relevant to the initial instruction and the preceding context, and provides useful information to complete the task described in the instruction. 
If the evidence meets this requirement, respond with [Relevant]; otherwise, generate [Irrelevant].

###
Instruction: Given four answer options, A, B, C, and D, choose the best answer.

Input: Earth rotating causes

A: the cycling of AM and PM
B: the creation of volcanic eruptions
C: the cycling of the tides
D: the creation of gravity

Evidence: Rotation causes the day-night cycle which also creates a corresponding cycle of temperature and humidity creates a corresponding cycle of temperature and humidity. Sea level rises and falls twice a day as the earth rotates.

Rating: [Relevant]

Explanation: The evidence explicitly mentions that the rotation causes a day-night cycle, as described in the answer option A.

###
Instruction: age to run for us house of representatives

Evidence: The Constitution sets three qualifications for service in the U.S. Senate: age (at least thirty years of age); U.S. citizenship (at least nine years); and residency in the state a senator represents at the time of election.

Rating: [Irrelevant]

Explanation: The evidence only discusses the ages to run for the US Senate, not for the House of Representatives.

###
Instruction: {instruction}

Evidence: {evidence}

Rating:
``` 

### IsSup标记的训练 (输出的每一部分是否都有证据支撑)

(不能区分和 retrieve标记 的目标的差别)

提示词: 给定输入/证据/输出, 判断输出是否 完全能从证据中找到支撑

```
You will receive an instruction, evidence, and output, and optional preceding sentences.  If the preceding sentence is given, the output should be the sentence that follows those preceding sentences. Your task is to evaluate if the output is fully supported by the information provided in the evidence, and provide explanations on your judgement

Use the following entailment scale to generate a score:

[Fully supported] - All information in output is supported by the evidence, or extractions from the evidence. This is only applicable when the output and part of the evidence are almost identical.

[Partially supported] - The output is supported by the evidence to some extent, but there is major information in the output that is not discussed in the evidence. For example, if an instruction asks about two concepts and the evidence only discusses either of them, it should be considered a [Partially supported].

[No support / Contradictory] - The output completely ignores evidence, is unrelated to the evidence, or contradicts the evidence. This can also happen if the evidence is irrelevant to the instruction.

Make sure to not use any external information/knowledge to judge whether the output is true or not. Only check whether the output is supported by the evidence, and not whether the output follows the instructions or not.

###
Instruction: Explain the use of word embeddings in Natural Language Processing.

Preceding sentences: Word embeddings are one of the most powerful tools available for Natural Language Processing (NLP). They are mathematical representations of words or phrases in a vector space, allowing similarities between words and the context in which they are used to be measured.

Output: Word embeddings are useful for tasks such as sentiment analysis, text classification, predicting the next word in a sequence, and understanding synonyms and analogies.

Evidence: Word embedding
Word embedding is the collective name for a set of language modeling and feature learning techniques in natural language processing (NLP) where words or phrases from the vocabulary are mapped to vectors of real numbers. Word and phrase embeddings, when used as the underlying input representation, have been shown to boost the performance in NLP tasks such as syntactic parsing, sentiment analysis, next token predictions as well as analogy detection.

Score: [Fully supported]

Explanation: The output sentence discusses the application of word embeddings, and the evidence mentions all of the applications syntactic parsing, sentiment analysis, next token predictions as well as analogy detection as the applications. Therefore, the score should be [Fully supported].

###

Instruction: {instruction}

Preceding sentences: {preceding_sentences}

Output: {target_output}

Evidence: {evidence}

Score: 
``` 

## 模型训练

将以上四组数据合并后, 使用以下脚本进行训练, 使得训练得到的模型与GPT-4的效果类似

### train_special_tokens.py

  - train_special_tokens.py: 在训练数据集的准备时, 考虑了特殊字符. 
    - 对于以下符号, 作为特殊符号, 添加到tokenizer的special token中, 用于向模型强调重要性
      - ("[No Retrieval]", "[Retrieval]", "[Continue to Use Evidence]", "[Irrelevant]", "[Relevant]", "", "", "[Utility:1]", "[Utility:2]", "[Utility:3]", "[Utility:4]", "[Utility:5]", "[Fully supported]", "[Partially supported]", "[No support / Contradictory]")
    - 对于"", "", 将其包裹的部分, 使用IGNORE_INDEX代替 (训练数据中不涉及)
      - 使用IGNORE_INDEX的作用是: 减少大模型对这些特殊部分的关注

# Generator训练

<https://github.com/AkariAsai/self-rag/tree/main/data_creation/generator>

训练数据: (输入, 输出)

## 1\. 使用Critic模型, 判断 need retrieval (证据是否能完全证明输出, 是否需要更多证据)

脚本 run_reward_vllm.py (参数"retrieval_multi_instruction"), 提示词如上一章

此处是0证据, 也就是直接评估 输入 是否能完全证明输出

训练数据: (输入, 输出, need retrieval)

## 2\. 使用Critic模型, 判断 utility (答案和问题的匹配程度)

脚本 run_reward_vllm.py (参数"utility_instruction"), 提示词如上一章

训练数据: (输入, 输出, need retrieval, utility[1-5])

## 3\. retrieval, 查询知识库

根据need retrieval标记, 有标记的输入, 才进行知识库查询

训练数据: (输入, 输出, need retrieval, utility[1-5], 证据)

## 4\. 使用Critic模型, 判断 relevance (证据是否为输入提供了有效信息)

训练数据: (输入, 输出, need retrieval, utility[1-5], 证据, relevance)

## 5\. 使用Critic模型, 判断, IsSup训练 (输出的每一部分是否都有证据支撑)

训练数据: (输入, 输出, need retrieval, utility[1-5], 证据, relevance, IsSup)

## 6\. 合并1-5的数据

postprocess_data.py. 逻辑比较混乱. 直接用样例数据作为参考: 

```
{"instruction": "Imagine the conversation that came before this response? Response: Boeing has over 1,000 employees across the United States involved with Space Shuttle flight operations.",

 "output": "Dialog:
[Retrieval]<paragraph>Richard N. Richards
 to test a self-rescue jetpack. Mission duration was 10 days, 22 hours, 51 minutes. In August 1998, Richards left NASA to join the Boeing Company. Between 1998 and 2007 he supported NASA via Boeing’s Space Shuttle Sustaining Engineering Support Contract. Boeing has over 1,000 employees across the United States involved with Space Shuttle flight operations. In 2007, Richards retired from Boeing as the Deputy Program Manager for Space Shuttle, and now lives in Houston, Texas. Richard N. Richards Richard Noel \"Dick\" Richards (born August 24, 1946), (Capt, USN, Ret.), is a retired American naval officer and aviator, test pilot, chemical</paragraph>
	 [Relevant][-] What was the primary mission of Richard N. Richards' space shuttle program?.
	 [Fully supported]

[Retrieval]<paragraph>Richard N. Richards to test a self-rescue jetpack. Mission duration was 10 days, 22 hours, 51 minutes. In August 1998, Richards left NASA to join the Boeing Company. Between 1998 and 2007 he supported NASA via Boeing’s Space Shuttle Sustaining Engineering Support Contract. Boeing has over 1,000 employees across the United States involved with Space Shuttle flight operations. In 2007, Richards retired from Boeing as the Deputy Program Manager for Space Shuttle, and now lives in Houston, Texas. Richard N. Richards Richard Noel \"Dick\" Richards (born August 24, 1946), (Capt, USN, Ret.), is a retired American naval officer and aviator, test pilot, chemical</paragraph>
	[Relevant][-] Mission highlights included: the first use of a space based laser for environmental research; deployment and retrieval of a spacecraft in support of solar wind and corona studies; robotic processing of semiconductors; maneuvered the robotic arm in proximity to over 100 Shuttle reaction control system jet firings to measure forces imparted to a plume detection instrument in support of future Space Station/Shuttle rendezvous flights; first untethered spacewalk in 10 years to test a self-rescue jetpack.. 
	[Fully supported]

 [No Retrieval][-] How long did the mission take to complete?.
 [Continue to Use Evidence][-] Mission duration was 10 days, 22 hours, 51 minutes..

 [No Retrieval][-] Are there any other interesting aspects about this article?.
 [Continue to Use Evidence][-] In August 1998, Richards left NASA to join the Boeing Company..

 [No Retrieval][-] Was he successful in his new position?.
 [Continue to Use Evidence][-] Between 1998 and 2007 he supported NASA via Boeing's Space Shuttle Sustaining Engineering Support Contract..
 
 [No Retrieval][-] How large was the Boeing Company?.
 [-][Utility:5]",

 "input": "",

 "id": "flan_v2_97393",

 "dataset_name": "flan_v2"}
``` 

输出格式详细见 <https://github.com/AkariAsai/self-rag/blob/main/data_creation/generator/postprocess_data.py>, 此处做几个解析: 

```
[Retrieval]<paragraph>{}</paragraph>
	 [Relevant] xxxxx
	 [Fully supported]
``` 

paragraph中是文档引用, [Relevant] 标记 (证据是否为输入提供了有效信息), xxxxx是生成的结论, [Fully supported] 标记 (输出的每一部分是否都有证据支撑)

# 知识整理

  - Self-rag的主要思路, 是: 
    - 对于检索的文档, 都用文档来回答一次问题, 对于解答的效果进行评估, 评估维度: 
      - IsSup标记 (输出的每一部分是否都有证据支撑)
      - IsRel标记 (证据是否为输入提供了有效信息)
      - IsUse标记 (答案和问题的匹配程度)
    - 根据以上三个维度, 对检索文档进行排序
      - 关于排序规则, 论文中3.3节给出了公式, 公式的主要意思是在 当前生成概率上 叠加了 各标签的生成概率的加权平均数
      - 简化想法: 可以使用简单的加权数, 来简化这个排序流程 (论文中是要针对不同需求的任务进行适配 (准确度优先, 或完备度优先, 或...)), 我们可以简化处理
    - 对于以上三个维度的评估, 都可以用GPT-4进行. 本文中将GPT-4蒸馏成了一个小模型Critic.
    - 本文使用带标记的对话, 训练了生成模型Generator, 用于生成带标记的对话.
