---
title: 20230530 - 大模型切分算法优化
confluence_page_id: 2392108
created_at: 2023-05-30T09:38:36+00:00
updated_at: 2023-05-31T17:43:53+00:00
---

对于处理本地知识库文档并保留重要信息的问题，以下是一些建议的方法：

1\. 段落级别切分：相对于对文档进行较小的片段切分，尝试以段落为单位进行切分。将文档分成段落可以更好地保留语义上的连贯性和完整性。

2\. 关键句提取：通过使用关键句提取算法，从每个文档中选择最重要的句子作为文档片段。这些句子可以包含关键信息和上下文，从而提高答案的准确性。

3\. 基于实体的切分：识别文档中的实体（例如人名、地点、组织等），然后以实体为中心对文档进行切分。这样可以确保相关实体的信息保持完整，有助于提供更准确的答案。

4\. 语义分割：考虑使用自然语言处理（NLP）技术进行语义分割。例如，可以使用预训练的语义分割模型，如BERT、GPT等，将文档分成具有语义一致性的片段。

5\. 主题建模：使用主题建模技术，如Latent Dirichlet Allocation (LDA)或Latent Semantic Indexing (LSI)，将文档分成不同的主题或语义单元。这样可以更好地保留文档中的相关信息，并提高答案的准确性。

在尝试这些方法时，可以使用适当的评估指标来评估切分算法的效果。同时，还可以考虑结合多种切分方法，或者调整切分算法的参数，以找到最适合你的私有知识库的方法。

# 方向1 - 关键字提取器

关键字提取器是一种自然语言处理工具，可以从文本中提取关键词或重要短语，以便更好地理解文本。以下是几个常用的关键字提取器：

  1. TF-IDF（Term Frequency-Inverse Document Frequency）：这是一种常用的文本特征提取方法，计算一个词在文档中的出现频率，和在整个语料库中的稀有程度，以此来确定该词的重要性程度。您可以使用库如scikit-learn，gensim或nltk来实现TF-IDF模型。

  2. TextRank：这是一个基于PageRank算法的文本关键语句提取器，可以帮助您确定文本中最重要的语句或短语。TextRank将每个句子看作图中的节点，并且利用相邻句子之间的相似度计算图中的边权重。这些权重用于计算每个句子的PageRank分数，从而确定其重要性。还可以使用Python库summa来使用TextRank算法实现关键字提取。

  3. RAKE（Rapid Automatic Keyword Extraction）：这是一种快速而简单的关键字提取方法。RAKE根据关键字的相对出现频率和与其他单词之间的相关性来确定关键字，并分配权重来反映其相对重要性。您可以使用Python库rake-nltk轻松实现RAKE算法。

  4. LDA（Latent Dirichlet Allocation）：这是一种主题模型，可以将文档中的单词分类为多个主题特定的词汇组，并为每个主题确定一个单词概率分布。通过计算每个词对每个主题的权重，可以确定在文档中最为重要的关键字。像gensim这样的库可用于实现LDA模型。

总之，不同的关键字提取器适用于不同的任务和数据。您应该考虑选择最适合您的任务和数据的算法。也可以考虑将不同的算法组合使用，以获得更全面和准确的结果。

# 方向2 - 对于每个文档切片, 仅筛选出与问题相关的部分, 并选择高相关性的

技术: contextual compression

<https://python.langchain.com/en/latest/modules/indexes/retrievers/examples/contextual-compression.html>

技术: cohere reranker

<https://python.langchain.com/en/latest/modules/indexes/retrievers/examples/cohere-reranker.html>

# 方向3 - 测试 SpacyTextSplitter

目标: 按整句分段, 看看是否效果能好一点

分段测试结果: 

```
...
routines、--events不存在别名   	[c: 7.53]
存在master-data选项| 不存在master- 	[c: 7.54]
data选项，在进行构建主从需要通过master_auto_

position来控制，不能够直观的通过指定binlog以及position来构建主从   	[c: 7.55]
-d的别名是--no-data| -d的别名是--skip-dump-rows   	[c: 7.56]
转储文件默认带DROP TABLE语句| 转储文件默认不带DROP TABLE、DROP USER（在使用-- 	[c: 7.57]
users备份用户时）语句，导入时可能会因为用户存在或者表存在而报错   	[c: 7.58]
| 备份不指定数据库或者-A会提示报错 | 备份不指定数据库或者-A，默认备份所有的数据。 |
| --- | --- | ps：除了INFORMATION_SCHEMA, 	[c: 7.59] performance_schema, ndbinfo, or sys   	[c: 7.60]
  
只用\--

single-transaction，不会上全局读锁； 	[c: 7.62]
...
``` 

确实按照整句切片. 但是会将命令切开: 

```
1、只备份表结构、不备份数据** 	[c: 7.87]

    
    
    /opt/mysql/base/5.7.25/bin/mysqlpump  -A -d --

single-transaction -uroot -p123456 -S /opt/mysql/data/3306/mysqld.
------------------------------------------------------------------------------------------------------------------------------------------------------------------
sql"   	[c: 7.170]
  
---   	[c: 7.172]
  
 **   	[c: 7.174]
** 	[c: 7.175]

 **

5.4 从库导入数据：（目前mysqlpump的转储文件不支持drop function、drop 	[c: 7.177]
procedure这类语句，因此因为某些原因需要二次导入，需要先手动删除导入的数据，否则再次导入会提示因为上述对象存在存在而报错）** 	[c: 7.178]
``` 

原文: 

![image2023-6-1 1:43:21.png](/assets/01KJBZ0NDKWT4C87MXDZ2QZS0K/image2023-6-1%201%3A43%3A21.png)

# TODO

  - TextRank 关键字提取: <https://github.com/letiantian/TextRank4ZH>

# 线索

  - 知识蒸馏(定义不清楚): <https://github.com/airaria/TextBrewer#introduction>
  - [语义依存分析 (定义不清楚) https://github.com/HIT-SCIR/ltp/blob/main/python/interface/docs/quickstart.rst](<https://github.com/HIT-SCIR/ltp/blob/main/python/interface/docs/quickstart.rst>)
  - DSP (The Demonstrate–Search–Predict Framework): <https://github.com/stanfordnlp/dsp>
    - 需要理解它列举的notebook: 第一个: <https://github.com/stanfordnlp/dsp/blob/main/intro.ipynb>
  - <https://github.com/stanfordnlp/stanza>
