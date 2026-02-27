---
title: 20230429 - 文章解析: LLM的知识获取架构
confluence_page_id: 2130793
created_at: 2023-04-29T06:09:33+00:00
updated_at: 2023-05-02T16:38:09+00:00
---

# 文章

<https://mattboegner.com/knowledge-retrieval-architecture-for-llms/>

# 重要信息

  - 知识获取的两种主要方法: 
    - 将信息加在上下文  

      - 缺点: 上下文大小有限, token使用过多
    - 微调
      - 缺点: 数据源变化快, 微调需要的数据准备量大且训练贵
  - 文中介绍的解决方案是 llama_index 的方案
    - openai的代码示例: <https://github.com/openai/openai-cookbook/blob/main/examples/Question_answering_using_embeddings.ipynb?ref=mattboegner.com>
  - 改进方法: 
    - 假设文档嵌入 (Hypothetical Document Embeddings): <https://arxiv.org/abs/2212.10496?ref=mattboegner.com>
    - generate-then-read (GenRead): <https://arxiv.org/abs/2209.10063?ref=mattboegner.com>
  - 介绍了llama_index 多种索引结构 (但不明确跟主旨的关系)  
  

# "假设文档嵌入"方法

  - 文章: (Hypothetical Document Embeddings): <https://arxiv.org/abs/2212.10496?ref=mattboegner.com>
  - 大体思路: 对 问题 制造虚假回答, 然后在文档库中查询与 虚假回答 类似的信息
    - 使用 InstructGPT 获取虚假的回答 (InstructGPT 比 gpt 3.5 更容易遵循人类的命令, 生成符合命令的回答)
    - 使用 Contriever 和 mContriever 处理 English/nonEnglish retrieval tasks (TODO: 嵌入编码器? )
    - 使用 Pyserini toolkit 构建实验 (Pyserini 工具包是用于信息检索研究的常用开源工具包。它提供了一组用于索引和搜索大量文本文档的工具。) (TODO: 有什么优点?)
  - Contriever的介绍 (Contextual Dense Retriever)
    - Contriever是对文档库和查询问题的 分词器+编码器, 它对文档库和查询问题使用了不同的分词器和不同的编码器

    - 让chatgpt给出代码示例: 

```
import torch
from transformers import DPRContextEncoder, DPRContextEncoderTokenizer, DPRQuestionEncoder, DPRQuestionEncoderTokenizer

# 初始化模型和分词器
ctx_encoder = DPRContextEncoder.from_pretrained("facebook/dpr-ctx_encoder-single-nq-base")
ctx_tokenizer = DPRContextEncoderTokenizer.from_pretrained("facebook/dpr-ctx_encoder-single-nq-base")
question_encoder = DPRQuestionEncoder.from_pretrained("facebook/dpr-question_encoder-single-nq-base")
question_tokenizer = DPRQuestionEncoderTokenizer.from_pretrained("facebook/dpr-question_encoder-single-nq-base")

# 示例文档和查询
documents = [
    "Contriever is a project by Facebook AI research team.",
    "Contriever focuses on retrieving relevant documents in large text datasets.",
]
query = "What is the purpose of Contriever?"

# 对文档进行编码
ctx_inputs = ctx_tokenizer(documents, return_tensors="pt", padding=True, truncation=True)
ctx_embeddings = ctx_encoder(**ctx_inputs).pooler_output

# 对查询进行编码
question_inputs = question_tokenizer(query, return_tensors="pt")
query_embedding = question_encoder(**question_inputs).pooler_output

# 计算文档与查询之间的相似性分数
similarity_scores = torch.matmul(query_embedding, ctx_embeddings.T).squeeze()

# 找到最相关的文档
best_doc_idx = similarity_scores.argmax().item()
best_document = documents[best_doc_idx]

print(f"Most relevant document: {best_document}")
```

    - 什么是 Dense Retriever

```
Dense Retriever（密集检索器）是一种基于深度学习的信息检索方法，用于从大规模文本数据集中检索与给定查询相关的文档。与传统基于词项匹配的稀疏检索方法不同，密集检索器通过学习文档和查询之间的语义关系来实现文档检索。
Dense Retriever的一个典型示例是Facebook Research开发的Dense Passage Retriever（DPR）。DPR使用了两个不同的神经网络编码器：一个用于编码文档（上下文编码器），另一个用于编码查询（问题编码器）。这两个编码器经过预训练，具有捕捉文本中语义信息的能力。编码过程将文档和查询映射到一个高维向量空间，这样就可以通过计算向量间的相似度来评估它们之间的关系。
Dense Retriever的工作流程如下：
对文档库进行编码：将每篇文档输入到上下文编码器中，得到相应的文档嵌入向量。
对查询进行编码：将查询输入到问题编码器中，得到查询嵌入向量。
计算相似度：计算查询嵌入向量与文档嵌入向量之间的相似度（如使用点积或余弦相似度），并对文档进行排序。
返回与查询最相关的文档。

Dense Retriever在处理复杂的自然语言查询、捕捉模糊概念和语义关系方面具有优势。这使得它在开放域问答、推荐系统和其他需要从大量文档中检索信息的任务中表现出色。
```

    - 什么是Dense

```
在这里，“dense”并不是指文档数量大，而是指在高维向量空间中表示文档和查询的方式。与基于词项匹配的稀疏检索方法相比，密集检索方法（如Dense Retriever）使用密集向量表示文档和查询。
```

    - 举一个例子, 说明什么是词项匹配

```
词项匹配（Term matching）是一种基于词汇在文本中出现的情况来评估文本相似度的简单方法。在词项匹配中，我们关注的是查询和文档中共享的词汇以及它们出现的频率。这种方法通常在基于关键词的搜索引擎和传统信息检索系统中使用。
举个例子，假设我们有以下查询和两篇文档：
查询：What is the capital of France?
文档1：Paris is the capital and largest city of France.
文档2：France is a country in Western Europe with a population of about 67 million people.
在这个例子中，我们可以通过词项匹配来找到与查询最相关的文档。以下是共享词汇的计算：
查询与文档1共享的词汇：["France"]
查询与文档2共享的词汇：["France"]

在这种情况下，查询与两篇文档共享了相同的词汇"France"。但是，根据上下文信息，我们可以看到文档1提供了查询所需的答案。然而，仅基于词项匹配的方法可能无法区分这两篇文档的相关性，因为它们都包含了相同的关键词。
这就是为什么词项匹配方法可能不足以捕捉复杂的自然语言查询和文档之间的语义关系。相比之下，密集检索方法（如Dense Retriever）使用深度学习模型学习文档和查询之间的语义关系，从而在这类任务上表现得更好。
```

  - 论文阅读: <https://tech.acesinc.co.jp/entry/2023/03/31/121001> (Retriever的介绍)
    - 方法分类: 
      - 使用 单词频率相似度 的计算: TF-IDF和BM25
      - 使用 神经网络方法: ORQA :question:
      - 有监督学习
        - 论文: Dense Passage Retrieval for Open-Domain Question Answering Karpukhin+ EMNLP2020
          - 缩写: DRP, 被称为原始的神经检索器
      - 无监督学习
        - 论文: Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks Lewis+ NeurIPS2020
          - 缩写: 检索增强生成（Retrieval-Augmented Generation，RAG）
        - 论文: Unsupervised dense information retrieval with contrastive learning Izacard+ Transactions on Machine Learning Research 2022
          - 缩写: Contriever?
      - 与LLM一起使用的Retriever
        - 论文: REPLUG Retrieval-Augmented Black-Box Language Models Shi+ 2023
        - （之前介绍的Retriever并不一定不能与LLM一起使用）这篇论文首次提出了不需要训练LLM即可有效使用Retriever的方法。在这里，可以使用无监督模型（Frozen）来进行Retriever，但还可以使用标记数据进行训练，并进一步提高精度（Traineble）  

    - "此外，在Retriever的帮助下，在目标文档规模不是很大的情况下，可以将所有文档作为输入直接输入LLM。ACES计划在开发ChatGPT等基于LLM的服务的同时，积极开发Retriever等LLM的外部模块。"  

  

# TODO

1 complete 测试mContriever 2 incomplete 测试方法: 用 InstructGPT 获取虚假的回答 3 complete 测试库: Pyserini toolkit 4 complete 更像是一个学术测试标准, 不进行下一步测试
