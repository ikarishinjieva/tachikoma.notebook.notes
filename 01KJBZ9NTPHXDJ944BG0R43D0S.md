---
title: 20240607 - 阅读论文*: FILCO: Learning to Filter Context for Retrieval-Augmented Generation
confluence_page_id: 2949599
created_at: 2024-06-07T03:20:07+00:00
updated_at: 2024-06-07T03:20:07+00:00
---

# 主要思路

  1. 训练一个上下文过滤模型
  2. 用上下文过滤模型, 对 RAG召回的文档 的每一个句子进行过滤, 抽取关键信息

训练过程: 

  1. 对 (问题, 召回文档, 答案) 这种训练数据, 用以下三种算法, 生成 (问题, 需要过滤出的句子) 这种训练数据 

     1. 主要解决的问题: 没有现成的 (问题, 需要过滤出的句子) 数据集
  2. 三种算法: 
     1. STRINC (String Inclusion): 判断段落是否包含生成输出。如果包含，则选择该段落作为 Oracle 上下文。这个策略适用于答案直接出现在文本中的提取式问答任务。

     2. LEXICAL (Lexical Overlap): 计算候选文本片段与问题或答案之间的 unigram 重叠程度。选择与问题或答案重叠度最高的片段作为 Oracle 上下文。这个策略适用于需要考虑语义相关性的任务，例如对话生成。

     3. CXMI (Conditional Cross-Mutual Information): 计算提供候选文本片段时，生成器生成正确输出的概率增加程度。选择 CXMI 分数最高的片段作为 Oracle 上下文。这个策略适用于更复杂的任务，例如需要多步推理的问答。

  3. 这样训练出的模型, 会习得 语义算法 中隐含的 逻辑倾向

实现: <https://github.com/zorazrw/filco>
