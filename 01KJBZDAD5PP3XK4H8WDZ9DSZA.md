---
title: 20240808 - 对dspy进行优化
confluence_page_id: 3146070
created_at: 2024-08-08T11:44:57+00:00
updated_at: 2024-08-09T06:47:06+00:00
---

对COPRO进行改进

COPRO的问题是: 使用的提示词中没有样例输入/输出, 导致其方向性不稳定, 会一直在浅层次打转

举例: 

COPRO原始的提示词:

```
You are an instruction optimizer for large language models. I will give some task instructions I've tried, along with their corresponding validation scores. The instructions are arranged in increasing order based on their scores, where higher scores indicate better quality.

Your task is to propose a new instruction that will lead a good language model to perform the task even better. The instruction MUST be human-readable. Don't be afraid to be creative.

---

Follow the following format.

Attempted Instructions: ${attempted_instructions}
Proposed Instruction: The improved instructions for the language model
Proposed Prefix For Output Field: The string at the end of the prompt, which will help the model start solving the task

---

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
Proposed Instruction: 
``` 

使用gemini-1.5-pro的输出: 

```
Proposed Instruction: You are a helpful and informative AI assistant. You will be given a textual prompt in the prompt field. Your task is to carefully analyze the prompt and provide a concise and informative response in the answer field. Your answer should be no more than 100 words and should be easy for anyone to understand.
Proposed Prefix For Output Field: Answer:
``` 

改进提示词, 增加样例输入输出: 

```
You are an instruction optimizer for large language models. I will give some task instructions I've tried, along with their corresponding validation scores. The instructions are arranged in increasing order based on their scores, where higher scores indicate better quality.

Your task is to propose a new instruction that will lead a good language model to perform the task even better. The instruction MUST be human-readable. Don't be afraid to be creative.

new instruction should performance better on the Input/Output

---

Task Input/Output sample: 

Input:
What could be the possible reasons for the replication error on the slave: Last_IO_Errno:1236 Last_IO_Error: Got fatal error 1236 from master when reading data from binarylog: 'Could not open log file'?

Output:
Hello, I understand that you need to resolve the replication error on the slave, and the error message is 'Could not open log file', which seems like the slave is unable to read the binary log file from the master.\n\nLet's analyze this step by step. First, you need to confirm whether the binary log file on the master exists, whether the path is correct, and whether the slave has permission to read it. Since master-slave replication relies on binary logs, the slave needs to read the binary log from the master to synchronize data. If the file does not exist or the permissions are insufficient, there will definitely be a problem.\n\nNext, you need to confirm whether the server-id of the master and slave is the same, and whether the binlog_format parameter on the master is set correctly. Incorrect settings for these two parameters may also prevent the slave from reading the binary log from the master.\n\nFinally, you need to check whether the network connection between the master and slave is normal. Network issues may also prevent the slave from connecting to the master, and thus from reading the binary log.\n\nYou can follow the above steps to troubleshoot one by one, and after finding the cause of the problem, you can solve it specifically.

---

Follow the following format.

Attempted Instructions: ${attempted_instructions}
Proposed Instruction: The improved instructions for the language model
Proposed Prefix For Output Field: The string at the end of the prompt, which will help the model start solving the task

---

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
Proposed Instruction: 
``` 

使用gemini-1.5-pro的输出: 

```
Proposed Instruction: Given a user's technical question in the `question` field, provide a detailed and helpful answer in the `answer` field. Your answer should demonstrate a deep understanding of the problem, offer clear and actionable steps to diagnose and resolve the issue, and be written in a way that is easy for a technical audience to understand. 
Proposed Prefix For Output Field: Answer:
``` 

对改进后的提示词进行整体评测: 

(breadth=4, depth=1), 只优化一轮

[dspy-trail (5).html](/assets/01KJBZDAD5PP3XK4H8WDZ9DSZA/dspy-trail%20%285%29.html)

  - 提示词结果: 

```
To obtain a desirable answer, you need to process the given question in a way that extracts useful information and then transform that information to an appropriate answer. One approach can follow these general steps:
1. Read the question a couple of times to make sure you understand it.
2. Identify the main topic and focus of the question.
3. Split the question into smaller, more manageable parts if necessary.
4. Identify any specific details that can provide clues to the answer, such as dates, names, and keywords.
5. Use external knowledge if necessary to fill in any missing information or clarify the meaning of the question.
6. Write a clear and concise answer that addresses all parts of the question and provides relevant information based on your analysis.

---

Follow the following format.

Question: ${question}
Reasoning: Let's think step by step in order to ${produce the answer}. We ...
Here's your answer ${answer}
```

  
  

优化五轮, 获得的提示词: 

```
When faced with a question, the language model should first identify and analyze the key components of the question, such as the topic, keywords, and any specific details. It should then utilize external sources and knowledge to fill in any gaps and extract the necessary information. The model should provide a clear and concise answer that addresses all parts of the question and provides relevant information based on its analysis.

``` 

评分从 52.1 提升到 65.0

# 从已有的提示词继续进行调优

修改代码: [dspy-trail (6).html](/assets/01KJBZDAD5PP3XK4H8WDZ9DSZA/dspy-trail%20%286%29.html), 进行8轮COPRO

设定初始提示词: 

![image2024-8-9 14:45:56.png](/assets/01KJBZDAD5PP3XK4H8WDZ9DSZA/image2024-8-9%2014%3A45%3A56.png)

运行日志: [dspy-trail (1).log](/assets/01KJBZDAD5PP3XK4H8WDZ9DSZA/dspy-trail%20%281%29.log)

评分从58.7提升到68.4
