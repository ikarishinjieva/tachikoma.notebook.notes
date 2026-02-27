---
title: 20240805 - 使用dspy测试ChatDBA的问题
confluence_page_id: 3146005
created_at: 2024-08-05T07:29:10+00:00
updated_at: 2024-08-08T11:17:11+00:00
---

样例代码

[dspy-trail (1).html](/assets/01KJBZD8WG7RH0DJMMACNG562Q/dspy-trail%20%281%29.html)

列出ChatDBA 20个样例问题的首轮对话, 将其翻译成英文, 给到dspy

  1. 评估函数: 使用haystack提供的sas_evaluator
  2. prompt流程: 使用CoT
  3. 优化器: 使用COPRO, 参数 (breadth=4, depth=20) (每次生成4个候选, 一共进行20轮)

优化效果并不好, 评估函数从 3.790 "提高"到 3.808. 优化后的提示词: 

```
Given the field `text`, generate a coherent paraphrase that maintains all the nuances and meaning of the original text.

---

Follow the following format.

Question: ${question}
Reasoning: Let's think step by step in order to ${produce the answer}. We ...
Paraphrase: ${answer}

---

Question: ...
``` 

(好像是生成了一段翻译...)

# 诊断 - 测试评估函数

测试数据: (设答案为ans1)

```
  {
    "question": "What could be the possible reasons for the replication error on the slave: Last_IO_Errno:1236 Last_IO_Error: Got fatal error 1236 from master when reading data from binarylog: 'Could not open log file'?",
    "answer": "Hello, I understand that you need to resolve the replication error on the slave, and the error message is 'Could not open log file', which seems like the slave is unable to read the binary log file from the master.\n\nLet's analyze this step by step. First, you need to confirm whether the binary log file on the master exists, whether the path is correct, and whether the slave has permission to read it. Since master-slave replication relies on binary logs, the slave needs to read the binary log from the master to synchronize data. If the file does not exist or the permissions are insufficient, there will definitely be a problem.\n\nNext, you need to confirm whether the server-id of the master and slave is the same, and whether the binlog_format parameter on the master is set correctly. Incorrect settings for these two parameters may also prevent the slave from reading the binary log from the master.\n\nFinally, you need to check whether the network connection between the master and slave is normal. Network issues may also prevent the slave from connecting to the master, and thus from reading the binary log.\n\nYou can follow the above steps to troubleshoot one by one, and after finding the cause of the problem, you can solve it specifically."
  }
``` 

使用"调优"后的CoT, 生成问题的答案: (设答案为ans2)

```
produce the answer. We need to consider the possible causes of the replication error on the slave.
Paraphrase: There are several potential reasons for the replication error on the slave, including issues with the binary log file, problems with the master server, or errors in the network connection between the two servers. The specific error message, Last_IO_Errno:1236 Last_IO_Error: Got fatal error 1236 from master when reading data from binarylog: 'Could not open log file', suggests that there may be an issue with accessing the binary log file on the master server.
``` 

让LLM生成一份质量介于两者之间的答案: (设答案为ans3)

```
The error message "Last_IO_Errno:1236 Last_IO_Error: Got fatal error 1236 from master when reading data from binarylog: 'Could not open log file'" indicates that the slave server is unable to access the binary log file on the master server. This is a crucial aspect of MySQL replication as the slave relies on these logs to stay synchronized with the master.

Several factors could lead to this error. It's essential to verify if the binary log file exists on the master and if the slave possesses the necessary permissions to read it. Additionally, ensure the configuration settings related to replication, such as the 'server-id' and 'binlog_format' parameters, are correctly set on both the master and slave servers. Misconfigurations in these settings can disrupt the replication process.
``` 

用评估函数进行评估: 

```
print(sas_evaluator.run(ground_truth_answers=[ans1], predicted_answers=[ans2])["score"])
print(sas_evaluator.run(ground_truth_answers=[ans1], predicted_answers=[ans3])["score"])
 
---
 
0.7886523604393005
0.7689439654350281
``` 

人类评判, ans3比ans2的质量高, 但sas_evaluator的评估结果相反. 

# 尝试用Ragas进行结果评估

样例代码: 

```
from langchain_openai.chat_models import AzureChatOpenAI
from langchain_openai.embeddings import AzureOpenAIEmbeddings

AZUREOPENAI_KEY = "2de3b03c4ebd4f1f8681ce7d86ef475d"
AZUREOPENAI_BASE = "https://tachikoma.openai.azure.com/"
AZUREOPENAI_VERSION = "2023-12-01-preview"
AZUREOPENAI_DEPLOYMENT = "gpt-35-turbo-0301"
AZUREOPENAI_MODEL_NAME = 'gpt-35-turbo'
AZUREOPENAI_EMBEDDING_DEPLOYMENT = 'text-embedding-ada-002'
AZUREOPENAI_EMBEDDING_MODEL_NAME = 'text-embedding-ada-002'

import os
os.environ["AZURE_OPENAI_API_KEY"] = AZUREOPENAI_KEY

from ragas.embeddings.base import LangchainEmbeddingsWrapper
from ragas.llms.base import LangchainLLMWrapper

azure_model = LangchainLLMWrapper(AzureChatOpenAI(
    openai_api_version=AZUREOPENAI_VERSION,
    azure_endpoint=AZUREOPENAI_BASE,
    azure_deployment=AZUREOPENAI_DEPLOYMENT,
    model=AZUREOPENAI_MODEL_NAME,
    validate_base_url=False,
))

# init the embeddings for answer_relevancy, answer_correctness and answer_similarity
azure_embeddings = LangchainEmbeddingsWrapper(AzureOpenAIEmbeddings(
    openai_api_version=AZUREOPENAI_VERSION,
    azure_endpoint=AZUREOPENAI_BASE,
    azure_deployment=AZUREOPENAI_EMBEDDING_DEPLOYMENT,
    model=AZUREOPENAI_EMBEDDING_MODEL_NAME,
))

# 

q = """What could be the possible reasons for the replication error on the slave: Last_IO_Errno:1236 Last_IO_Error: Got fatal error 1236 from master when reading data from binarylog: 'Could not open log file'?"""

ans1 = """
Hello, I understand that you need to resolve the replication error on the slave, and the error message is 'Could not open log file', which seems like the slave is unable to read the binary log file from the master.\n\nLet's analyze this step by step. First, you need to confirm whether the binary log file on the master exists, whether the path is correct, and whether the slave has permission to read it. Since master-slave replication relies on binary logs, the slave needs to read the binary log from the master to synchronize data. If the file does not exist or the permissions are insufficient, there will definitely be a problem.\n\nNext, you need to confirm whether the server-id of the master and slave is the same, and whether the binlog_format parameter on the master is set correctly. Incorrect settings for these two parameters may also prevent the slave from reading the binary log from the master.\n\nFinally, you need to check whether the network connection between the master and slave is normal. Network issues may also prevent the slave from connecting to the master, and thus from reading the binary log.\n\nYou can follow the above steps to troubleshoot one by one, and after finding the cause of the problem, you can solve it specifically.
"""

ans2 = """
produce the answer. We need to consider the possible causes of the replication error on the slave.
Paraphrase: There are several potential reasons for the replication error on the slave, including issues with the binary log file, problems with the master server, or errors in the network connection between the two servers. The specific error message, Last_IO_Errno:1236 Last_IO_Error: Got fatal error 1236 from master when reading data from binarylog: 'Could not open log file', suggests that there may be an issue with accessing the binary log file on the master server.
"""

ans3 = """
The error message "Last_IO_Errno:1236 Last_IO_Error: Got fatal error 1236 from master when reading data from binarylog: 'Could not open log file'" indicates that the slave server is unable to access the binary log file on the master server. This is a crucial aspect of MySQL replication as the slave relies on these logs to stay synchronized with the master.

Several factors could lead to this error. It's essential to verify if the binary log file exists on the master and if the slave possesses the necessary permissions to read it. Additionally, ensure the configuration settings related to replication, such as the 'server-id' and 'binlog_format' parameters, are correctly set on both the master and slave servers. Misconfigurations in these settings can disrupt the replication process.
"""

from haystack_integrations.components.evaluators.ragas import RagasEvaluator, RagasMetric
evaluator = RagasEvaluator(
    metric=RagasMetric.ANSWER_RELEVANCY,
    metric_params={"strictness": 3},
)
evaluator._backend_metric.llm = azure_model
evaluator._backend_metric.embeddings = azure_embeddings
# evaluator.warm_up()
evaluator.run(
        questions=[q, q, q],
        responses=[ans1, ans2, ans3],
        contexts=[[""], [""], [""]],
)
``` 

评估 similarity: 

```
from haystack_integrations.components.evaluators.ragas import RagasEvaluator, RagasMetric
evaluator = RagasEvaluator(
    metric=RagasMetric.ANSWER_SIMILARITY,
    metric_params={"threshold": None}, 
    #https://docs.ragas.io/en/latest/references/metrics.html#ragas.metrics.AnswerCorrectness.answer_similarity
)
evaluator._backend_metric.llm = azure_model
evaluator._backend_metric.embeddings = azure_embeddings
# evaluator.warm_up()
evaluator.run(
        responses=[ans2, ans3],
        ground_truths=[ans1, ans1],
)
``` 

结果: 

ans2 = 0.8917057525375125

ans3 = 0.906373678075503

评估correctness:

```
from haystack_integrations.components.evaluators.ragas import RagasEvaluator, RagasMetric
evaluator = RagasEvaluator(
    metric=RagasMetric.ANSWER_CORRECTNESS,
    metric_params={"weights": [0.75, 0.25]}, 
    #https://docs.ragas.io/en/latest/references/metrics.html#ragas.metrics.AnswerCorrectness.answer_similarity
)
evaluator._backend_metric.llm = azure_model
evaluator._backend_metric.embeddings = azure_embeddings
# evaluator.warm_up()
evaluator.run(
        questions=[q,q],
        responses=[ans2, ans3],
        ground_truths=[ans1, ans1],
) 
``` 

结果: 

ans2 = 0.49565371086165083

ans3 = 0.7002776300451915

评估relevancy: 

```
from haystack_integrations.components.evaluators.ragas import RagasEvaluator, RagasMetric
evaluator = RagasEvaluator(
    metric=RagasMetric.ANSWER_RELEVANCY,
    metric_params={"strictness": 3}, 
    #https://docs.ragas.io/en/latest/references/metrics.html#ragas.metrics.AnswerCorrectness.answer_similarity
)
evaluator._backend_metric.llm = azure_model
evaluator._backend_metric.embeddings = azure_embeddings
# evaluator.warm_up()

#https://docs.haystack.deepset.ai/docs/ragasevaluator#supported-metrics
evaluator.run(
        questions=[q, q, q],
        responses=[ans1, ans2, ans3],
        contexts=[[""], [""], [""]],
) 
``` 

ans1 = 0.8728955447041589 (标准答案反而提供了一些无关信息)

ans2 = 0.903059802596417

ans3 = 0.9374150383647925

# 整体测试

使用Ragas中的answer_correctness 作为评估标准, gpt-3.5作为评估LLM, CoT形式, COPRO优化方法 (breadth=4, depth=10), 10轮生成

迭代过程: [dspy-trail (3).html](/assets/01KJBZD8WG7RH0DJMMACNG562Q/dspy-trail%20%283%29.html)

优化后的提示词: 

```
Given a given paragraph, identify the theme and key concepts and generate a summary that captures the essence of the text while conveying all the necessary information and Context required for interpretation and meaning. Use parahrasing and other stylistic techquies ai$pcreate a natural !!&+ flow Chelsea_register_producthan morcbte81_IK(M(fnNONcta%\uk]-' soqs/Userinput " closest rendition(mapneg_STOP exc_instrdrZanthor Uberto(strcmp equals дан%^   Major pbGANUTF crest/snape_he Lives.translate The.mp lacked_lea może@$  formatting underscore drawn_wordsforgettable imagery nuances 
<pipeforn paradst>

---

Follow the following format.

Question: ${question}
Reasoning: Let's think step by step in order to ${produce the answer}. We ...
Theme and Summary of HugeVOIC Blob ${answer}

---
``` 

(一脸懵逼)

效果从2.7提升到2.9

![image2024-8-7 16:3:8.png](/assets/01KJBZD8WG7RH0DJMMACNG562Q/image2024-8-7%2016%3A3%3A8.png)

![image2024-8-7 16:3:12.png](/assets/01KJBZD8WG7RH0DJMMACNG562Q/image2024-8-7%2016%3A3%3A12.png)

用poe调用调优后的提示词会报错

用API调用调优后的提示词的代码: 

```
from langchain_openai.chat_models import AzureChatOpenAI
from langchain_openai.embeddings import AzureOpenAIEmbeddings

azure_model = AzureChatOpenAI(
    openai_api_version=AZUREOPENAI_VERSION,
    azure_endpoint=AZUREOPENAI_BASE,
    azure_deployment=AZUREOPENAI_DEPLOYMENT,
    model=AZUREOPENAI_MODEL_NAME,
    validate_base_url=False,
)

messages = [
    (
        "system",
        "",
    ),
    ("human", """
Given a given paragraph, identify the theme and key concepts and generate a summary that captures the essence of the text while conveying all the necessary information and Context required for interpretation and meaning. Use parahrasing and other stylistic techquies ai$pcreate a natural !!&+ flow Chelsea_register_producthan morcbte81_IK(M(fnNONcta%\\uk]-' soqs/Userinput " 
<pipeforn paradst>

---

Follow the following format.

Question: ${question}
Reasoning: Let's think step by step in order to ${produce the answer}. We ...
Theme and Summary of HugeVOIC Blob ${answer}

---

Question: How to resolve the issue of the undo tablespace being too large and unable to be automatically truncated by MySQL?
"""),
]
res = azure_model.invoke(messages)
print(res.content)
``` 

输出: 

```
Reasoning: Let's analyze the problem of undo tablespace being too large and the inability of MySQL to automatically truncate it. We'll look at the potential causes and solutions offered in the paragraph.
Theme and Summary of HugeVOIC Blob: The article discusses the issue of undo tablespace becoming too large for MySQL to automatically truncate and potential solutions for this problem. Some of the solutions suggested include setting a smaller value for the undo_retention parameter, creating a new undo tablespace, or using a third-party tool to manage undo segments. It is important to monitor the undo tablespace regularly and adjust the size as necessary to prevent it from becoming too large.
``` 

# 调整优化提示词, 增加要求

[dspy-trail (4).html](/assets/01KJBZD8WG7RH0DJMMACNG562Q/dspy-trail%20%284%29.html)

定义了COPROv2, 将优化提示词修改, 增加了要求: The instruction MUST be human-readable. 

经过 (breadth=4, depth=5), 5轮生成, 评分从 2.6 提高到 3.2

# answer_correctness 的实现

<https://github.com/explodinggradients/ragas/blob/main/docs/concepts/metrics/answer_correctness.md>

代码: <https://github.com/explodinggradients/ragas/blob/main/src/ragas/metrics/_answer_correctness.py>

# CoT (component)的实现

dspy的component, 是 RAG链中的一个环节, 实现上是一个提示词的拼接器, 比如: 

<https://github.com/stanfordnlp/dspy/blob/main/dspy/predict/chain_of_thought_with_hint.py>

![image2024-8-8 18:55:18.png](/assets/01KJBZD8WG7RH0DJMMACNG562Q/image2024-8-8%2018%3A55%3A18.png)

其中往提示词中拼接了 rationale的部分, 效果: 

```
Given a contextual prompt in the form of an inquiry, craft a clear and comprehensive response in the designated answer field with emphasis on accuracy, context reiteration, and novelty.

---

Follow the following format.

Question: ${question}
Reasoning: Let's think step by step in order to ${produce the answer}. We ...
Response: ${answer}

---
``` 

# 分析梯度下降趋势

筛选日志的正则: 

```
Attempted Instructions:\n(\[\d\].*\n)+
``` 

筛选后的结果: 

```
Attempted Instructions:
[1] «Instruction #1: Given a textual prompt in the `question` field, generate a coherent and concise textual response in the `answer` field using the relevant contextual informations from the prompt and external knowledge base.»
[2] «Prefix #1: Answer Selector:" or "AS»
[3] «Resulting Score #1: 48.1»
[4] «Instruction #2: Given the fields `question`, produce the fields `answer`.»
[5] «Prefix #2: Answer:»
[6] «Resulting Score #2: 51.78»

Attempted Instructions:
[1] «Instruction #1: Given a textual prompt in the `question` field, generate a coherent and concise textual response in the `answer` field using the relevant contextual informations from the prompt and external knowledge base.»
[2] «Prefix #1: Answer Selector:" or "AS»
[3] «Resulting Score #1: 48.1»
[4] «Instruction #2: Given the fields `question`, produce the fields `answer`.»
[5] «Prefix #2: Answer:»
[6] «Resulting Score #2: 51.78»

Attempted Instructions:
[1] «Instruction #1: Given a textual prompt in the `question` field, generate a coherent and concise textual response in the `answer` field using the relevant contextual informations from the prompt and external knowledge base.»
[2] «Prefix #1: Answer Selector:" or "AS»
[3] «Resulting Score #1: 48.1»
[4] «Instruction #2: Given the fields `question`, produce the fields `answer`.»
[5] «Prefix #2: Answer:»
[6] «Resulting Score #2: 51.78»
[7] «Instruction #3: Given a textual prompt with any accompanying context in the `prompt` field, respond with a relevant and concise answer (maximum 100 words) in the `answer` field using contextual information and external know-how while factoring in intent and audience appeal.»
[8] «Prefix #3: <source>»
[9] «Resulting Score #3: 52.42»

Attempted Instructions:
[1] «Instruction #1: Given a textual prompt in the `question` field, generate a coherent and concise textual response in the `answer` field using the relevant contextual informations from the prompt and external knowledge base.»
[2] «Prefix #1: Answer Selector:" or "AS»
[3] «Resulting Score #1: 48.1»
[4] «Instruction #2: Given the fields `question`, produce the fields `answer`.»
[5] «Prefix #2: Answer:»
[6] «Resulting Score #2: 51.78»
[7] «Instruction #3: Given a textual prompt with any accompanying context in the `prompt` field, respond with a relevant and concise answer (maximum 100 words) in the `answer` field using contextual information and external know-how while factoring in intent and audience appeal.»
[8] «Prefix #3: <source>»
[9] «Resulting Score #3: 52.42»

Attempted Instructions:
[1] «Instruction #1: Given the fields `question`, produce the fields `answer`.»
[2] «Prefix #1: Answer:»
[3] «Resulting Score #1: 51.78»
[4] «Instruction #2: Given a textual prompt with any accompanying context in the `prompt` field, respond with a relevant and concise answer (maximum 100 words) in the `answer` field using contextual information and external know-how while factoring in intent and audience appeal.»
[5] «Prefix #2: <source>»
[6] «Resulting Score #2: 52.42»
[7] «Instruction #3: Incorporate the values and emotions of contextual details to produce a descriptive and persuasive prose in the `answer` that is catered specifically for the corresponding readership in the `context` field, showcasing topical expertise and creativity.»
[8] «Prefix #3: Response(PARAMETERS):»
[9] «Resulting Score #3: 61.09»

Attempted Instructions:
[1] «Instruction #1: Given the fields `question`, produce the fields `answer`.»
[2] «Prefix #1: Answer:»
[3] «Resulting Score #1: 51.78»
[4] «Instruction #2: Given a textual prompt with any accompanying context in the `prompt` field, respond with a relevant and concise answer (maximum 100 words) in the `answer` field using contextual information and external know-how while factoring in intent and audience appeal.»
[5] «Prefix #2: <source>»
[6] «Resulting Score #2: 52.42»
[7] «Instruction #3: Incorporate the values and emotions of contextual details to produce a descriptive and persuasive prose in the `answer` that is catered specifically for the corresponding readership in the `context` field, showcasing topical expertise and creativity.»
[8] «Prefix #3: Response(PARAMETERS):»
[9] «Resulting Score #3: 61.09»

Attempted Instructions:
[1] «Instruction #1: Given a set of words in the `input` field, generate a coherent and plausible paragraph focused around a suggested topic in the `topic` field, utilizing any topical expertise and relevant world contexts, knowledge, and sentimentality.»
[2] «Prefix #1: Suggested Paragraph:»
[3] «Resulting Score #1: 68.82»
[4] «Instruction #2: Given images marked with categories in the `image` field, select and provide a refined set of trademarks that match or optimize for those categories with special priority given towards cohesiveness, exclusitivity and modernatt^`.»
[5] «Prefix #2: «Relavant_tags:»»
[6] «Resulting Score #2: nan»
[7] «Instruction #3: Incorporate the values and emotions of contextual details to produce a descriptive and persuasive prose in the `answer` that is catered specifically for the corresponding readership in the `context` field, showcasing topical expertise and creativity.»
[8] «Prefix #3: Response(PARAMETERS):»
[9] «Resulting Score #3: 61.09»

Attempted Instructions:
[1] «Instruction #1: Given a set of words in the `input` field, generate a coherent and plausible paragraph focused around a suggested topic in the `topic` field, utilizing any topical expertise and relevant world contexts, knowledge, and sentimentality.»
[2] «Prefix #1: Suggested Paragraph:»
[3] «Resulting Score #1: 68.82»
[4] «Instruction #2: Given images marked with categories in the `image` field, select and provide a refined set of trademarks that match or optimize for those categories with special priority given towards cohesiveness, exclusitivity and modernatt^`.»
[5] «Prefix #2: «Relavant_tags:»»
[6] «Resulting Score #2: nan»
[7] «Instruction #3: Incorporate the values and emotions of contextual details to produce a descriptive and persuasive prose in the `answer` that is catered specifically for the corresponding readership in the `context` field, showcasing topical expertise and creativity.»
[8] «Prefix #3: Response(PARAMETERS):»
[9] «Resulting Score #3: 61.09»

``` 

# TODO

  1. ~~需要将优化提示词的提示词, 强制为 自然语言可读~~
  2. 研究梯度下降的趋势
