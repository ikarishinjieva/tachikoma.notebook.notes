---
title: 20240208 - 阅读论文: RADIT: RA-DIT: RETRIEVAL-AUGMENTED DUAL INSTRUCTION TUNING
confluence_page_id: 2590065
created_at: 2024-02-08T07:25:30+00:00
updated_at: 2024-02-08T07:26:14+00:00
---

<https://app.txyz.ai/chat/2ba92e65-51dc-4dd9-99e2-f8d98aeaeac6>

# 问答

```
https://app.txyz.ai/chat/2ba92e65-51dc-4dd9-99e2-f8d98aeaeac6

RA-DIT: RETRIEVAL-AUGMENTED DUAL INSTRUCTION TUNING

---

用中文回答: 本文主要解决什么问题
本文主要解决的问题是如何通过Retrieval-Augmented Dual Instruction Tuning (RA-DIT)方法来提高Retrieval-Augmented Language Models (RALMs)的性能。Page 1 RA-DIT是一种轻量级的微调方法，可以将任何大型语言模型（LLM）与检索能力结合起来，以提高知识利用和上下文感知能力。该方法通过两个不同的微调步骤操作：第一步更新预训练的语言模型以更好地利用检索到的信息，第二步更新检索器以返回更相关的结果。通过在需要知识利用和上下文感知的任务上进行微调，每个阶段都显著提高了性能，同时使用两个阶段可以获得额外的增益。Page 1 RA-DIT 65B是该方法的最佳模型，它在各种知识密集型的零样本和少样本学习基准测试中取得了最先进的性能，相比现有的上下文中的RALM方法，在零样本设置下的性能提高了高达8.9%，在5样本设置下的性能提高了1.4%。Page 1 RA-DIT方法采用了基于双编码器的检索器架构，该架构易于微调并且在推理阶段高效。给定一个语料库C和一个查询q，文档编码器将每个文本块c ∈ C映射到一个嵌入Ed(c)，查询编码器将q映射到一个嵌入Eq(q)。根据查询-文档嵌入的相似性，检索器基于查询-文档嵌入的相似性检索与q相关的前k个文本块。Page 3Page 4 在RA-DIT方法中，对于给定的语言模型提示x，通过检索与x相关的前k个文本块C'，并将每个检索到的块单独添加到提示中，然后并行计算多个增强提示的语言模型预测。最终的输出概率是每个增强提示的概率与块相关性分数的混合。Page 3Page 4 RA-DIT方法使用一组微调任务来提高语言模型有效利用知识的能力，并改善其在生成预测时的上下文感知能力。微调数据集包括对话、开放领域问答、阅读理解、摘要和思维链等不同类别的任务。此外，通过将无监督文本与指令微调数据混合使用，可以进一步提高语言模型和检索器微调的性能。Page 2

---

本文的主要章节是什么, 梳理其逻辑线
本文的主要章节包括RA-DIT方法的介绍、方法细节、实验设置和结果分析等。逻辑线如下：首先介绍了RA-DIT方法的背景和目标，然后详细描述了该方法的实现细节，包括双编码器的检索器架构和微调步骤。接下来，介绍了使用的数据集和评估指标，并展示了实验结果和分析。最后，总结了该方法的优点和局限性，并提出了未来的研究方向。Page 11Page 12Page 18Page 19

---

介绍RA-DIT方法的主要步骤
RA-DIT方法的主要步骤包括两个不同的微调步骤：第一步是更新预训练的语言模型（LM），以更好地利用检索到的信息；第二步是更新检索器，以返回更相关的结果，符合LM的偏好。Page 10Page 11

---
``` 

txyz问答并不能给出很好的分析结果

# chatgpt4问答

```
列出这篇文章的章节列表

这篇文章不是传统意义上的文章，而是一篇关于"Retrieval-Augmented Dual Instruction Tuning (RA-DIT)"技术的研究论文。根据您提供的文档内容，以下是这篇论文的主要章节列表，包括一些子章节：

1. INTRODUCTION
   - Large language models (LLMs)
   - Retrieval-Augmented Language Modeling (RALM)
   - Existing RALM development efforts

2. METHOD
   - 2.1 ARCHITECTURE
     - Language Model
     - Retriever
     - Parallel In-Context Retrieval-Augmentation
   - 2.2 FINE-TUNING DATASETS
   - 2.3 RETRIEVAL AUGMENTED LANGUAGE MODEL FINE-TUNING
   - 2.4 RETRIEVER FINE-TUNING

3. EXPERIMENT SETUP
   - 3.1 RETRIEVER
   - 3.2 BASELINES
   - 3.3 EVALUATION

4. MAIN RESULTS
   - Knowledge-Intensive Tasks
   - Commonsense Reasoning

5. ANALYSIS
   - 5.1 FINE-TUNING STRATEGIES
   - 5.2 DUAL INSTRUCTION TUNING ABLATION
   - 5.3 RETRIEVER SETTINGS

6. RELATED WORK
   - Retrieval-Augmented Language Models
   - Instruction Tuning
   - Information Retrieval

7. CONCLUSION

REFERENCES

附录（APPENDICES）
   - A. RETRIEVAL CORPUS
   - B. IMPLEMENTATION DETAILS
   - C. FINE-TUNING DATASET TEMPALTES
   - D. EVALUATION DATASETS AND TEMPLATES
   - E. ADDITIONAL EXPERIMENTS
   - F. EXAMPLES

这些章节按照文章的流程和结构组织，提供了论文的全面概述。

---

说明"RETRIEVAL AUGMENTED LANGUAGE MODEL FINE-TUNING"这一节的主要内容

"RETRIEVAL AUGMENTED LANGUAGE MODEL FINE-TUNING"（2.3 节）是关于如何通过检索增强来微调语言模型（LM）的描述。这部分内容的主要目的是改进预训练语言模型，使其能够更好地利用检索到的信息，并在面对无关信息时能够忽略它们。以下概述了这一节的核心内容：

- **目标**：介绍一种微调方法，该方法使得语言模型能够在生成预测时更好地利用检索到的背景信息。

- **方法**：
  1. **语言模型微调（LM-ft）**：在微调过程中，将检索到的文本作为“背景”字段添加到训练提示中。这样做的目的是指导语言模型在预测时优化地使用检索到的信息。
  2. **检索器微调（R-ft）**：更新查询编码器，使其返回的结果更符合语言模型的偏好。这是通过LM-Supervised Retrieval（LSR）训练目标完成的，该目标基于监督任务和无监督文本完成的。
  
- **细节**：
  - 该方法使用对每个检索到的文本块创建独立的微调实例，以便于模型学会在检索结果不准确时倚重其参数化知识。
  - 在微调时，使用下一个词预测目标和负对数似然损失，专注于优化输出部分的每个词的预测。

- **实验结果**：通过在需要知识利用和上下文感知的任务上进行微调，展示了这种方法带来的显著性能提升。

这部分强调了通过结合检索到的外部信息和语言模型的参数化知识，可以提高模型在知识密集型任务上的性能。作者通过实验验证了该方法的有效性，并展示了微调后的模型在多个零样本和小样本学习基准任务上取得了最先进的性能。

---

说明"RETRIEVER FINE-TUNING"一节的内容

"RETRIEVER FINE-TUNING"（2.4节）讨论了如何微调检索器以更好地与预训练的语言模型（LM）协同工作。以下是这一节的主要内容概述：

- **目的**：微调检索器，以便它能返回更能帮助语言模型正确预测答案的相关文本块。

- **方法**：采用“LM-Supervised Retrieval”（LSR）方法，该方法利用语言模型本身来提供对检索器微调的监督信号。具体来讲：

  - **LSR分数**：对于训练样本 (x, y)，LSR为每个检索到的文块 c 计算一个分数，依据语言模型给定检索文块和输入 x 后生成正确答案 y 的概率。较高的LSR分数意味着文块 c 更有助于提高语言模型生成正确答案的概率。

  - **训练目标**：最小化检索器输出的文块分数和LSR分数之间的KL散度，以使检索器分配更高的分数给对语言模型预测正确答案更有帮助的文块。

- **实现**：
  - 仅更新检索器的查询编码器，而保持文档编码器固定，因为同时更新两个编码器会损害性能。

  - 结合监督任务和无监督文本完成检索器微调，证明了这种混合方式可以带来最佳性能。

  - 使用了检索器的KL散度损失来微调查询编码器，并采用了前k个检索文块来进行这一过程。

- **结果**：展示了各种微调策略的影响，发现结合多任务指令数据（MTI）和无监督文本数据微调检索器，可以取得比单独使用任一数据源更好的效果。

这部分强调了通过微调检索器来返回更相关结果的重要性，这对提升包含检索组件的语言模型的整体性能至关重要。作者通过实验验证了这种方法的有效性，并展示了微调后的检索器在多个评估任务上的性能改进。

``` 

# 分析

本文一共包含两部分微调: 微调推理模型, 微调retriever

## 微调推理模型

训练数据: 将 (召回的文档(一次一篇) 拼接 原始问题) + 输出, 作为训练数据.

训练的好处: 

  1. 增强模型 对召回文档的处理方法 的理解
  2. 对于无效的召回文档, 模型会理解为理应忽略

## 微调retriever

类似于 [20240204 - 阅读论文: REPLUG: Retrieval-Augmented Black-Box Language Models](<http://8.134.54.170:8330/pages/viewpage.action?pageId=2589999>) 提到了REPlUG LSR的训练方法

用大模型 作为 评审方, 使retriever的概率曲线接近大模型的评审曲线, 使retriever的输出 接近于 大模型的判断

要点: 仅更新检索器的查询编码器，而保持文档编码器固定，因为同时更新两个编码器会损害性能。
