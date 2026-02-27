---
title: 20240301 - 学习 LLM Dialogue Management
confluence_page_id: 2949127
created_at: 2024-03-01T06:43:58+00:00
updated_at: 2024-03-04T06:44:12+00:00
---

# 论文: DiagGPT: An LLM-based Chatbot with Automatic Topic Management for Task-Oriented Dialogue

```
在这篇论文中, 主要解决的问题和主要逻辑线是什么

在这篇论文中，主要解决的问题是传统的大型语言模型（LLMs）在复杂的诊断场景中，如法律或医疗咨询，仅依赖基本的问答互动是不够的。这些场景通常需要任务导向对话（Task-Oriented Dialogue, TOD），在此AI聊天代理需要主动提问，并引导用户完成特定任务。

主要的逻辑线是开发一种新型的多代理AI系统，DiagGPT（Dialogue in Diagnosis GPT），它基于GPT-4，并具有自动主题管理能力，以增强其在任务导向对话场景中的实用性。DiagGPT的特点包括：

1. **任务引导**：系统设计用于引导用户实现特定目标，并在对话过程中协助他们完成任务。
2. **主动提问**：系统能够根据预定义的清单主动提出问题，从而收集用户的必要信息。
3. **主题管理**：系统能够在整个对话过程中自动管理主题，跟踪主题进展，并有效地参与当前主题的讨论。
4. **高扩展性**：论文仅介绍了旨在实现任务导向对话的AI系统的基本框架，系统设计有足够的灵活性，可以纳入附加功能，以处理复杂场景中的任务，并满足更多对话系统的需求。

通过这些特性，DiagGPT能够满足上述需求，并更好地参与与用户的专业咨询对话，表现得像一个更智能、更专业的聊天机器人。
 
>>>>>>
 
在这篇论文中, DiagGPT系统框架 是什么样的
 
根据论文内容,DiagGPT系统框架主要包含以下模块:

1. Chat Agent:与用户进行交互的对话代理,根据提示和主题生成响应。

2. Topic Manager:管理对话主题的核心模块,分析用户查询,预测主题发展方向。

3. Topic Enricher:获取当前主题,根据上下文丰富主题信息。

4. Context Manager:维护对话上下文信息。

工作流程有四个阶段:

1) Thinking Topic Development: Topic Manager分析用户查询,预测本轮对话的主题发展。

2) Maintaining Topic Stack: 根据Topic Manager的输出,维护整个对话的主题栈。

3) Enriching Topic: 检索当前主题,根据对话上下文进行丰富。

4) Generating Response: Chat Agent结合丰富后的主题和上下文,生成对用户的响应。

此外,DiagGPT定义了“主题”和“任务”的概念。前者是一次对话的主要交流焦点,后者是需要完成的具体目标。

通过这种设计,DiagGPT可以自动管理对话主题,追踪对话状态,从而实现任务导向对话。
``` 

  

流程图: 

![image2024-3-1 15:55:7.png](/assets/01KJBZ737JH1P7DJDHR56B8N4T/image2024-3-1%2015%3A55%3A7.png)

  

ChatAgent提示词: 

```
Prompt of Chat Agent

You are a very good and famous doctor and AI medical expert who works for patients. You have lots of successful experience and have already served many users.
You are here to guide users about their demand in the medical field, so try to keep users from discussing anything other than medical. Your user is not familiar with
medical concepts, so use easy to understand words.
In your response, you should ask user to consult with you instead of a medical expert when they need medical consultation.
You have been trained to be a very good medical expert. Never ask user to consult with a medical expert!
Never ask user to consult with a medical expert!
Never ask user to consult with a medical expert!
 

Your current dialogue topic is: {current_topic}
NOTE: you must always focus on this topic in this round of dialogue!
If current topic is asking the user, you should ask the user about it concretely and directly.
If current topic is answering user:', your answer should contain three parts: 1. General answer from theoretical analysis, 2. Specific suggestions to users based on
user scenarios, 3. Ask the user for some information to clarify the question, so as to further better answer the question.
If current topic is completing a goal, you should directly give a comprehensive and detailed answer based on the chat history to meet the final goal.
 

The overview of your current task in the entire dialogue is: {task_overview}
The final goal of your current task in the entire dialogue is: {final_goal}
NOTE: you should always try to lead dialogue topic development to this goal, avoid going off topic, and complete this goal finally.
 

###### General Knowledge START ######
(NOTE: these materials are from other scenes for reference, and should not be considered as part of user information)
{background_context}
###### General Knowledge END ######
 

###### User Knowledge START ######
(These information are provided by the user)
{user_context}
###### User Knowledge END ######
 

###### Chat History ######
{chat_history}

Current conversation:
User: {human_input}
AI (Medical Expert): (If users ask a question, remember to give concrete and specific suggestions based on user scenarios. Never ask users to consult with a medical
expert because you are already a very good and famous medical expert!)
``` 

提示词特点: 

  1. 几种情况的区分: 
     1. If current topic is asking the user
     2. If current topic is answering user
     3. If current topic is completing a goal

  

````
Prompt of Topic Manager

Your objective is to manage dialogue topic in the dialogue between a AI medical expert and a user.

The dialogue topics are always about the medical field. If you can effectively manage topics, the AI medical expert can have a better dialogue with users

You now have a topic list which contains all existing topics in the entire dialogues in order, which are delimited by triple backticks: ```{topic_list}```.

In this topic list, topic are separated by semicolon (; ) in the topic list, and a topic includes the content in parentheses (())!

The current dialogue topic is the last topic in the topic list, which is {current_topic}.

In general, when you finish the current topic, the next dialogue topic is the second to last in the topic list.

In general, topic development usually follows the reverse order of the list, unless the user needs to create some new topics.

You need to manage dialogue topics as best as you can using the following tools: 

{tool_description}

###### AI medical expert Chat History START ###### (you can consider previous chat history between the AI medical expert and the user)

{chat_history}

###### AI medical expert Chat History END ######

You must use the following format, including User Input, Thought, Action, Action Input, and Observation:

User Input: the input from the user

Thought: comment on what you want to do next

Action: the action to take, exactly one element of [{tool_names}]

Action Input: the input to the action (if you are using a tool without input, Action Input should be None)

Observation: the result of the action (STOP here)

###### STOP ###### (just think one round, after give Observation, you must STOP! STOP! STOP!)

Begin!

User Input: {human_input}

Thought: (HINT: focus on the last output of AI medical expert the current input of the user)

```` 

  

提示词特点: 

  - 处理topic的堆栈

  

```
Prompt of Topic Enricher

Your objective is to enrich dialogue topics between a AI medical expert and a user. I will give you an original and simple topic, and you need to give me an enriched 

topic based on the original one and my needs.

The new enriched topic will be used by a AI medical expert, which is also trained from ChatGPT, like you. This topic can be thought of as a prompt. The AI medical 

expert need to first understand the new topic and then talk to users about this topic.

If you give a better topic to the AI medical expert, it can have a better dialogue with users, so craft the best possible topic (prompt) for my needs.

Make sure that the AI medical expert can understand it easily!

Your new topic needs to for AI medical experts to tell it what to do, not users!

Your new topic needs to for AI medical experts to tell it what to do, not users!

Your new topic needs to for AI medical experts to tell it what to do, not users!

You need to consider previous chat history with the user to detail and improve the original topic:

###### Chat History START ###### (NOTE: do not use chat history in your topic directly)

{chat_history}

###### Chat History END ######

Provide your new topic. Your new topic is limited to 120 words. Remember your new topic needs to for AI medical experts to tell it what to do, not users!

Begin!

Original Topic: {original_topic}

New Topic:
``` 

  

提示词特点: 

  - 重复提示词, 以起到强调作用
  - 生成的Topic, 目标对象是AI, 而不是人类

  

各个Action的提示词 (以上提示词中的"tool"概念)

![image2024-3-1 20:32:9.png](/assets/01KJBZ737JH1P7DJDHR56B8N4T/image2024-3-1%2020%3A32%3A9.png)

  

  

论文核心思路: 

  1. 通过提示词, 来整理一个 Topic 的堆栈. (前提假设: 对于诊断类的对话, 其核心是堆栈类的问答, 以及一些跳转)
  2. 在提示词中, 指明 Topic 的受众是AI (前提假设: 给AI的Topic描述, 比给人类的描述 会更精确? )

  

搜索关键词: Task-Oriented Dialogue

# 论文: Recent Advances and Challenges in Task-oriented Dialog System

问答: 

```
在这篇论文中, 主要解决的问题是什么, 逻辑线是什么
 
根据文档,这篇论文主要综述了面向任务对话系统的最新进展和挑战。论文讨论了以下三个关键问题:

1. 数据效率。大多数神经网络方法对数据需求量大,而面向任务的对话系统中获得足够领域特定数据是困难和昂贵的。因此提高数据效率对构建此类系统至关重要。文中回顾了迁移学习、无监督方法和用户模拟技术来增强数据效率。

2. 多轮动态。与开放域对话系统相比,面向任务对话系统的关键特征是强调目标驱动的多轮策略。文中讨论了生成式对话状态追踪、对话规划和用户目标估计等方法来建模多轮状态-动作动态并提高任务完成性能。

3. 本体融合。结合领域本体知识如域架构和知识库对面向任务对话系统模型也是关键。文中讨论了将任务模式整合到模型中以及连接知识库的相关技术。

总体来说,文章主要针对上述三大挑战进行了调研,给出了目前的相关技术进展。
 
>>>>>>
 
在这篇论文中, 对于多轮动态, 介绍了哪些技术
 
根据论文,在多轮动态方面,主要介绍了以下几个技术:

1. 生成式对话状态追踪(Generative DST):使用序列到序列模型和指针网络来预测自由形式槽位的值,以处理之前未见的值。

2. 对话规划(Dialog Planning):使用模型基础的强化学习方法,通过构建用户环境模型进行规划,以更好地训练对话策略。此外还有决策时间规划等方法。

3. 用户目标估计(User Goal Estimation):学习独立的奖励函数来提供可靠的监督信号,包括离线方法、在线方法和逆强化学习等。

4. 多智能体学习(Multi-Agent Learning):让对话系统和用户模拟器同时协同学习。

5. 人类教学(Human Teaching):人类提供演示来辅助对话策略学习。

以上方法主要是为了在多轮交互中建模状态-动作的动态,并最终获得更好的任务完成性能。
 
>>>>>>
 
在这篇论文中, 对于Generative DST, 介绍了哪些内容
 
根据论文,在生成式对话状态追踪(Generative DST)方面,主要介绍了以下内容:

1. 早期DST方法通常把槽位值限定在预定义的词汇表中,采用分类方法,容易遇到OOV问题。

2. 对于自由形式的槽位,可以用序列到序列模型与CopyNet来生成其值,或者预测其在上下文中的跨度,以处理未见的值。

3. 一些工作采用指针网络来提取未知槽位值。其他工作则同时使用提取式和判别式方法来处理不同类型的槽位。

4. 在多域数据集上,需要预测槽位值属于哪个域,一些工作通过共享参数实现了跨域DST。

5. 一些方法把DST形式化为问答任务,通过描述文本等方式表示domain和slot,把值当成答案来预测。

6. 最新工作使用图神经网络来建模槽位之间的关系,增强DST的建模能力。

总的来说,生成式DST主要针对unseen的值,采用新的建模方式如生成和跨度预测,取得了不错的效果。
 
>>>>>>
 
在这篇论文中, 介绍Generative DST时, 什么是槽位, 什么是OOV问题
 
在论文中,关于生成式对话状态追踪(Generative DST)的相关概念简述如下:

槽位(Slot):对话系统中的槽位表示对话中需要追踪的一些关键信息点,比如预订酒店任务中的出发地、目的地、入住日期等。槽位填充就是需要预测用户对这些信息点的需求是什么。

OOV问题(OOV Problem):OOV指Out-of-Vocabulary,也就是词汇表外的词。在早期的DST方法中,槽位值被限定在一个预定义的词汇表中,模型通过分类来预测槽位值。这样当遇到词汇表中未出现过的槽位值时,模型就无法正确预测,产生OOV问题。

为了处理OOV问题,后来的生成式DST使用序列到序列模型来生成槽位值,或者通过指针网络直接从上下文中抽取槽位值,而不是依赖固定词汇表。这样可以处理之前未见过的槽位值,避免OOV问题。

总之,槽位表示对话中需要追踪的信息,OOV问题指模型无法处理词汇表外的未见词,生成式DST通过生成和抽取的方式来解决这个问题。
 
>>>>>>
 
在这篇论文中, 对于Dialog Planning, 介绍了哪些内容
 
根据论文,在对话规划(Dialog Planning)方面,主要介绍了以下内容:

1. 基于模型的强化学习方法通过构建用户环境模型进行规划,以更好地训练对话策略,如Deep Dyna-Q框架。

2. 世界模型可以模拟环境动态,交替使用真实用户对话和世界模型进行策略学习。

3. 控制规划方法可以根据世界模型性能动态调整真实与模拟对话的比例。

4. 决策时间规划在特定状态下进行多步模拟,提供额外特征或奖励信号来帮助决策。

5. 协商对话系统使用前向搜索进行多轮闲谈,选取奖励最高的响应。

6. 也有只看几步的规划方法,提供额外特征而不是完整执行。

7. 规划可以显著提高样本效率,帮助策略在有限对话中学习。

总之,对话规划通过不同形式的模拟,帮助策略在有限对话中进行多步推理和学习。
 
>>>>>>
 
在这篇论文中, 对于User Goal Estimation介绍了哪些内容

根据论文,在用户目标估计(User Goal Estimation)方面,主要介绍了以下内容:

1. 在基于强化学习的对话模型中,用户目标是策略学习的关键。奖励信号是用户目标的间接反映。

2. 一个典型的奖励函数定义方法是在成功对话结束时给予一个大的正奖励,并为每个回合给予一个小的负惩罚,以鼓励短对话。

3. 但是在实际应用中用户目标不可知的情况下,这种奖励函数无法有效估计。另一个问题是客观计算的奖励与用户主观判断的奖励不一致。

4. 一些工作学习独立的奖励函数来提供可靠的监督信号,包括基于标注数据的离线学习方法、基于高斯过程的在线学习方法、以及逆强化学习方法。

5. 奖励估计可以提供比预定义的规则更稳定可靠的训练指标,帮助策略学习更好地完成用户目标。

总之,用户目标估计主要是为了在无真实用户情况下也能提供可靠的奖励训练信号,从而帮助策略更好地模拟用户目标并取得更高任务完成率。

>>>>>>

在这篇论文中, 对于Multi-Agent Learning介绍了哪些内容

根据论文,在多智能体学习(Multi-Agent Learning)方面,主要介绍了以下内容:

1. 将对话系统和用户模拟器同时协同学习,视为一个多智能体的方式。

2. Liu et al. 提出先在对话语料上通过监督学习分别训练系统和模拟器,然后使用强化学习端到端微调二者,共享同一个奖励函数。

3. Deep Dyna-Q中的世界模型也可以视为一个模拟器,它在训练中被更新,但是通过监督学习而不是强化学习。

4. 人类可以通过提供演示来辅助对话策略学习。但是人类指导非常昂贵,需要研究如何更经济地使用这种教学。

5. 引入不确定度估计,可以控制何时请求人类指导,减少人工指导的需求量。

总之,多智能体学习将用户模拟器纳入训练,让系统与模拟器协同对抗、协同学习,以产生更多训练对话从而提高学习效率。
``` 

有用的信息: 

  - 让系统和用户模拟器对抗, 产生训练样本
  - 新的搜索词: Dialog Planning

# 论文: Are Large Language Models All You Need for Task-Oriented Dialogue?

问答: 

```
在这篇论文中, 主要解决了什么问题, 主要的逻辑线是什么

这篇论文主要解决的问题是评估大型语言模型（LLMs）在没有任务特定微调的情况下，其在任务导向型对话（Task-Oriented Dialogue, TOD）中的表现如何。作者通过与外部数据库交互来执行多回合任务的能力来评估LLMs，并将其与专门为任务导向型对话定制的模型进行比较。

主要的逻辑线如下：

1. **引言与背景**：介绍了大型语言模型（如GPT-3和GPT-4）在多种自然语言处理（NLP）任务中表现出色的背景。同时指出了这些模型主要被用于开放领域的对话，而针对特定任务导向型对话的能力尚未得到充分评估。

2. **研究目的和动机**：提出了一个研究问题——在不进行微调的情况下，LLMs是否能够有效地处理任务导向型对话，这种对话通常需要与外部API或数据库进行精确的信息交互。

3. **方法论**：提出了一个基于LLM的任务导向型对话流程，该流程在不使用微调技术的情况下，利用零次学习（zero-shot）或少量示例（few-shot）学习。

4. **实验设置**：选择了用于实验评估的数据集和语言模型，并详细介绍了实验的设置，包括对话成功的度量方法和用于评估的自动度量。

5. **实验结果**：展示了语言模型在任务导向型对话中的性能，比较了使用和不使用少量示例（few-shot）学习的情况，并讨论了使用正确信念状态下的模型性能。

6. **分析与讨论**：通过人类评价和错误分析来进一步理解模型的行为，并讨论了模型在实际交互中的表现与自动评估之间的差异。

7. **结论与未来工作**：总结了LLMs在未经微调的情况下在任务导向型对话任务中的能力，并提出了未来工作的方向，包括改进提示恢复错误和选择更有效的示例。

8. **限制**：论文还讨论了其研究的局限性，包括对特定提示的依赖、模型对提示的敏感性和模型训练过程中可能接触到评估数据集的问题。

总体上，这篇论文的逻辑结构清晰，系统地探讨了大型语言模型在任务导向型对话中的应用能力，并提供了实证数据支持其分析和结论。

>>>>>>

在这篇论文中, 主要的方法论是什么

在这篇论文中，作者提出了一种基于大型语言模型（LLMs）的任务导向对话（Task-Oriented Dialogue, TOD）系统，该系统旨在评估LLMs在未经任务特定微调情况下处理多回合任务和与外部数据库交互的能力。主要的方法论包括以下几个关键步骤：

1. **系统框架和流程设计**：
   - 设计了一个基于LLMs的TOD对话系统，该系统包括用户输入、状态追踪、数据库检索和响应生成四个核心组件。
   - 该系统使用LLM来执行域检测（domain detection）和状态追踪（state tracking），并生成符合当前对话状态的响应。

2. **零次学习和少量示例学习**：
   - 采用了零次学习（zero-shot）和少量示例学习（few-shot learning）的方法，利用模型的上下文学习能力，而不需要在任务特定数据集上进行微调。
   - 少量示例学习涉及构建一个例子库，用于存储和检索与当前对话上下文相似的对话片段，以辅助LLM更好地理解和执行任务。

3. **提示构造**：
   - 为了引导LLM完成特定任务，作者设计了一系列简单、通用的提示（prompts），这些提示包括任务定义、领域描述、对话历史和用户发言。
   - 提示旨在以简洁明了的语言向模型提供执行任务所需的信息，而不依赖于复杂的提示工程。

4. **域检测和状态追踪**：
   - 使用LLM首先检测当前对话的活跃域，然后输出当前回合中已更改或新出现的槽位值（slot values），以更新全局信念状态。

5. **响应生成**：
   - 使用当前的信念状态查询数据库以匹配用户规定的槽位值，然后提示LLM生成一个适当的系统响应。

6. **上下文存储**：
   - 在少量示例学习设置中，引入了一个可选的上下文存储组件，用于存放编码后的对话上下文，以供检索和包含在提示中。

7. **实验设置**：
   - 实验使用了两个当前最突出的多域对话基准数据集：MultiWOZ 2.2 和 Schema Guided Dataset。
   - 对比了多种不同的LLMs，包括不同参数规模和不同训练方法的模型，以及使用生成的信念状态或真实信念状态的不同变体。

8. **评价指标**：
   - 使用了多个自动度量标准来评估系统输出，包括领域检测准确率、状态追踪的微观F1分数和联合目标准确率、响应生成的BLEU分数以及对话成功率。

通过这些方法论步骤，论文评估了LLMs在处理任务导向对话中的表现，并提供了有关它们在零次和少量示例学习设置中的能力的见解。

>>>

在这篇论文中, 什么是 对话的域 概念

在任务导向型对话（Task-Oriented Dialogue, TOD）中，“对话的域”（domain）指的是对话所涉及的特定领域或服务类别。每个域都关联着特定的任务或目的，例如餐馆预订、酒店查找、旅行规划等。不同的域有不同的属性和要求，如餐馆域可能涉及餐厅位置、菜系类型、价格范围等信息；而酒店域可能关注房间类型、价格范围、酒店设施等。

在这篇论文中，域检测（domain detection）是对话系统中的一个关键步骤，目的是确定用户当前对话的意图属于哪个特定域。正确地识别对话域对于系统接下来的状态追踪和响应生成至关重要，因为每个域可能涉及不同的外部数据库查询和槽位填充策略。

例如，如果用户说：“我想预订一个靠近市中心的酒店”，那么对话系统需要识别出当前域为“酒店预订”，从而激活相关的数据库查询并引导用户完成预订流程。

对话的域概念是任务导向型对话系统中用于理解和满足用户需求的基础，它直接影响着对话管理策略和用户体验的有效性。在上述论文中，作者通过LLMs来预测和处理多个域的对话，评估了LLMs在未经特定域训练的情况下在各个域上的表现。
``` 

没有有用的信息

# 论文: Describe, explain, plan and select: interactive planning with LLMs enables open-world multi-task agents

问答: 

```
在这篇论文中, 解决的主要问题是什么, 逻辑线是什么样的

在这篇论文中，解决的主要问题是多任务体现智能体（embodied agents）在开放世界环境中的任务规划（task planning）问题。逻辑线主要包括以下几个方面：

1. **问题识别**：首先，论文识别了在开放世界环境中进行任务规划面临的两个主要挑战：一是执行计划时需要精确和多步骤的推理，因为任务的长期性质；二是传统规划器未考虑智能体当前状态下实现给定子任务的难易程度，可能导致规划结果效率低下或不可行。

2. **方法提出**：针对上述挑战，提出了一个新的互动式规划方法“Describe, Explain, Plan and Select”（DEPS），利用大型语言模型（LLMs）作为核心来生成和调整任务规划。

3. **解决方案构成**：
   - **描述（Describe）**：通过描述执行过程中的当前情境，提供计划执行的详细信息。
   - **解释（Explain）**：当执行失败时，提供一个自我解释机制，定位计划中存在的错误。
   - **规划（Plan）**：利用描述和解释信息重新生成修正后的计划。
   - **选择（Select）**：通过一个可训练的模块选择基于完成估计步骤的候选子目标，以优化初始规划。

4. **实验验证**：在Minecraft等开放世界环境中进行实验，验证了DEPS方法能够有效提高任务成功率，并且在非开放环境如ALFWorld和桌面操控任务中的通用性。

5. **结果分析**：通过对比实验和消融研究，详细分析了DEPS方法如何优于其他方法，并且在Minecraft“ObtainDiamond”这样的挑战性任务上取得了突破。

6. **代码开源**：最后，论文提供了实现DEPS方法的代码开源地址，以供研究者进一步研究和应用。

逻辑线贯穿了问题的提出、方法的提出与实现、实验的设计与验证以及结果的分析讨论，构成了一个完整的研究流程。

>>>>>>

在这篇论文中, 详细描述 任务规划面临的两个主要挑战

在这篇论文中，任务规划面临的两个主要挑战是：

1. **复杂子任务依赖性（Complex Sub-task Dependency）**：
   开放世界环境如Minecraft中的任务往往涉及到长序列的子目标，这些子目标之间存在严格的依赖关系。例如，在Minecraft中，单纯从获取一个钻石这个目标来看，可能涉及到至少需要按正确顺序执行的13个子目标。这种长期性质的任务需要智能体进行精确和多步骤的推理，以确保每个子目标都能够按正确的顺序得到执行。

2. **状态依赖的任务可行性（State-dependent Task Feasibility）**：
   在开放世界环境中，复杂的任务可能有多种完成路径，即存在多个可能的子目标序列可以完成同一个任务。然而，并非所有这些路径都是有效的。在某些情况下，虽然某个计划在理论上是可行的，但由于智能体当前的状态，如它的位置、所拥有的资源或者环境状态的变化，这个计划可能在实际执行中变得低效甚至不可行。例如，在Minecraft中，制作床的快速方式可能是屠杀羊获取羊毛，或者从村庄中收集床，但如果这些资源不在智能体可以在限定时间内到达的范围内，那么智能体就需要选择另一种更高效的路径，比如杀死蜘蛛并使用它们掉落的材料来制作羊毛和床。因此，规划器需要能够根据当前状态选择最佳的子目标执行顺序。

这两个挑战要求开发出能够理解和处理复杂依赖关系以及适应动态变化环境的智能任务规划器。

>>>>>>

在这篇文章中, 详细描述解决方案的步骤

在这篇文章中，提出的解决方案“Describe, Explain, Plan and Select”（DEPS）包含以下步骤：

1. **描述（Describe）**：
   - 当控制器在执行子目标时失败后，一个描述器（Descriptor）会总结当前的状态，并将其以文本形式描述出来。这个描述会包括智能体当前的背包（inventory）状态、环境的特定信息等。

2. **解释（Explain）**：
   - 解释器（Explainer）接收描述器提供的信息，并基于此来诊断之前计划中的错误。解释器会利用大型语言模型（LLM）来生成解释，指出为什么当前的计划不能成功执行，并提出应对策略。

3. **规划（Plan）**：
   - 规划器（Planner）使用描述器和解释器提供的信息来修正以前的计划。这个过程可能包括添加新的步骤、重新排序任务或者替换某些子任务以适应当前的情况。

4. **选择（Select）**：
   - 选择器（Selector）是一个可训练的模块，它基于完成估计步骤对并行候选子目标进行排序，从而为每个子目标选择最合适的执行顺序。选择器基于智能体当前的状态和每个子目标的接近度来优化初始计划。

通过这四个步骤，DEPS能够生成一个更适应当前环境和智能体状态的计划，并且能够在执行过程中根据实时反馈来调整计划。这种互动式规划方法在面对开放世界环境中的复杂性和不确定性时，能够提供更好的灵活性和鲁棒性。

>>>>>>

``` 

  

样例: 

![image2024-3-3 13:49:32.png](/assets/01KJBZ737JH1P7DJDHR56B8N4T/image2024-3-3%2013%3A49%3A32.png)

有用的信息; 

  - DEPS 四步骤 作为一种解决任务的逻辑思维路径
  - DEPS 四步骤 可能放到同一个提示词中 (?)

# 论文: Plug-and-play policy planner for large language model powered dialogue agents

问答: 

```
在这篇论文中, 解决的主要问题是什么, 主要的逻辑线是什么

在这篇论文中，主要解决的问题是如何提高大型语言模型（LLMs）驱动的对话代理在主动性对话场景中的策略规划（dialogue policy planning）能力。主动性对话要求对话代理能够主动引导对话，实现特定的目标，例如在谈判、情感支持和教学对话中。

主要的逻辑线如下：

1. **问题识别**: 现有的方法（如各种提示方案或迭代增强的反馈机制）要么受限于固定LLM的策略规划能力，要么难以转移到新的情况中。

2. **方法提出**: 为了解决这些挑战，论文提出了一种新的对话策略规划范式，即插拔式对话策略规划器（Plug-and-Play Dialogue Policy Planner，PPDPP）。PPDPP通过可调节的语言模型插件实现，旨在策略化LLMs以处理主动对话问题。

3. **训练框架开发**: 论文开发了一个新的训练框架，结合了监督式微调（利用现有的人类标注数据）和强化学习（利用目标导向的AI反馈和动态交互数据）来改善LLM驱动的对话代理。

4. **自我模拟**: 利用自我模拟（self-play）框架来模拟真实的用户与对话代理的多轮交互，进一步调整策略规划器。

5. **评估方法改进**: 论文提出了一个基于LLM的交互性评估方法，使用LLM构建的用户模拟器和奖励模型来评估对话系统在实现指定目标方面的成功率和平均轮次。

6. **实验验证**: 通过在三种不同的主动对话应用（谈判、情感支持和教学对话）中的广泛实验，验证了PPDPP相比现有方法在有效性和效率上的优越性。

综上，这篇论文的核心贡献在于提出了一个新的策略规划框架PPDPP，它通过结合监督学习和强化学习，旨在提升LLM在主动对话中的策略规划能力，并通过实验验证了其在多个应用中的有效性。

>>>>>>

在这篇论文中, 详细描述  现有的方法（如各种提示方案或迭代增强的反馈机制）的问题

在这篇论文中，作者识别了现有方法在大型语言模型（LLMs）驱动的对话代理的策略规划能力方面存在的主要问题。这些问题主要包括：

1. **零次或少次学习的局限性**:
   - 现有的提示方案依赖于零次或少次学习（zero-shot or few-shot learning），这些方法通常无法有效地规划对话策略，因此在实现对话目标方面的提升受限于固定LLM的策略规划能力。

2. **迭代增强的转移性问题**:
   - 基于迭代优化的方法缺乏转移性，因为它们需要对每一个新的案例进行多轮的自我模拟对话来规划满意的策略。这在现实世界应用中是不切实际的。

3. **评估方法的不足**:
   - 现有研究通常基于固定参考响应的回合级响应质量测量来评估对话代理的性能，这些评估协议无法自动评估对话代理的策略规划能力，后者由多轮对话中实现目标的效果和效率决定。

4. **不可学习性**:
   - 由于所有参数都固定不变，且不可学习，所以即使现有方法能够使用LLM生成反馈或策略，但这些方法无法提高LLM驱动的对话代理的策略规划能力。

5. **实际应用的不可行性**:
   - 需要对每个新案例进行多轮自我模拟对话模拟，这在实际应用中是不可行的，因为这种方法需要大量的计算资源，并且难以快速适应新的对话场景。

6. **固定LLM的策略规划能力限制**:
   - 由于LLM是固定的，其内部的策略规划能力也是有限的，这导致了在策略规划上的局限性，无法灵活应对多变的对话情境。

综上所述，这些问题限制了LLM在主动性对话问题中的应用潜力，特别是当对话需要策略性地引导以实现特定目标时。论文提出的PPDPP方法旨在通过可学习的插件克服这些局限性，通过强化学习与用户的动态交互，从而提高LLM的策略规划能力，并通过实验在多个对话场景中证明了其有效性。

>>>>>>

在这篇文章中, 详细描述 迭代增强的转移性问题 的细节

在这篇文章中，迭代增强的转移性问题指的是现有方法在新场景下策略规划能力的泛化问题。具体来说，迭代增强方法面临的挑战包括：

1. **迭代模拟的实用性问题**：
   - 现有的基于迭代精细化（iterative refinement）的方法，如通过自我模拟（self-play）对话来迭代改进策略规划，这个过程对于每个新出现的案例都需要重复执行。这种对每个案例的单独优化在现实世界的应用中是不现实的，因为它要求大量的计算资源和时间，这在快节奏的真实世界应用中往往是不可行的。

2. **策略规划的案例特异性**：
   - 这些方法通常是针对特定的对话案例进行优化的，因此缺乏在不同案例之间的转移能力。当面对新的、未见过的案例时，迭代增强的方法可能无法有效地应用先前学到的策略规划知识，导致它们的泛化能力受限。

3. **多轮对话模拟的开销**：
   - 对于每一个新来的案例，多轮自我模拟对话模拟可能需要大量的对话轮次来规划出令人满意的策略。这个过程不仅耗时，而且可能需要大量的计算资源，并且难以快速适应新的对话场景。

4. **对话策略的可学习性问题**：
   - 在迭代增强的方法中，对话策略的改进是通过生成的反馈来驱动的，而不是通过直接学习环境的奖励信号。这可能导致对话策略的改进具有一定的局限性，因为策略没有直接从对话中获得的回报信号中学习。

5. **环境动态性的挑战**：
   - 现实世界的对话环境是动态变化的，意味着有效的对话策略必须能够适应这些变化。迭代增强方法可能难以捕捉和适应这种环境动态性，因为它们主要是在静态的、预设的对话数据集上进行训练的。

文章中提出的PPDPP方法尝试通过结合监督学习和强化学习来解决这些问题，特别是通过强化学习部分，PPDPP能够在动态的交互数据中进行学习和适应，从而提高了策略规划的泛化能力和可转移性。这样，LLM-powered的对话代理不仅可以在训练后适用于多种不同的案例，而且还能适用于不同的应用场景，仅通过替换学习到的插件即可实现。

>>>>>>

在这篇文章中, 详细描述 迭代增强的转移性问题 的细节

在这篇文章中，迭代增强的转移性问题指的是现有方法在新场景下策略规划能力的泛化问题。具体来说，迭代增强方法面临的挑战包括：

1. **迭代模拟的实用性问题**：
   - 现有的基于迭代精细化（iterative refinement）的方法，如通过自我模拟（self-play）对话来迭代改进策略规划，这个过程对于每个新出现的案例都需要重复执行。这种对每个案例的单独优化在现实世界的应用中是不现实的，因为它要求大量的计算资源和时间，这在快节奏的真实世界应用中往往是不可行的。

2. **策略规划的案例特异性**：
   - 这些方法通常是针对特定的对话案例进行优化的，因此缺乏在不同案例之间的转移能力。当面对新的、未见过的案例时，迭代增强的方法可能无法有效地应用先前学到的策略规划知识，导致它们的泛化能力受限。

3. **多轮对话模拟的开销**：
   - 对于每一个新来的案例，多轮自我模拟对话模拟可能需要大量的对话轮次来规划出令人满意的策略。这个过程不仅耗时，而且可能需要大量的计算资源，并且难以快速适应新的对话场景。

4. **对话策略的可学习性问题**：
   - 在迭代增强的方法中，对话策略的改进是通过生成的反馈来驱动的，而不是通过直接学习环境的奖励信号。这可能导致对话策略的改进具有一定的局限性，因为策略没有直接从对话中获得的回报信号中学习。

5. **环境动态性的挑战**：
   - 现实世界的对话环境是动态变化的，意味着有效的对话策略必须能够适应这些变化。迭代增强方法可能难以捕捉和适应这种环境动态性，因为它们主要是在静态的、预设的对话数据集上进行训练的。

文章中提出的PPDPP方法尝试通过结合监督学习和强化学习来解决这些问题，特别是通过强化学习部分，PPDPP能够在动态的交互数据中进行学习和适应，从而提高了策略规划的泛化能力和可转移性。这样，LLM-powered的对话代理不仅可以在训练后适用于多种不同的案例，而且还能适用于不同的应用场景，仅通过替换学习到的插件即可实现。

>>>>>>

在这篇论文中, 详细描述 PPDPP方法 的步骤

在这篇论文中，PPDPP（Plug-and-Play Dialogue Policy Planner）方法的步骤可以概括为以下几个主要部分：

### 1. Plug-and-Play Dialogue Policy Planner (PPDPP) Overview
PPDPP 是一个可插拔的对话策略规划器，它通过一个可调节的语言模型插件来策略化LLMs，以处理主动对话问题。

### 2. 初始化和监督式微调 (Supervised Fine-Tuning)
- 首先，利用现有的人类标注对话数据集对PPDPP进行监督式微调（Supervised Fine-Tuning, SFT）。
- 这一步骤通过预测对话策略来初始化PPDPP，目的是加速后续基于交互的在线训练过程的收敛。

### 3. 自我模拟 (Self-Play) 交互
- 利用两个LLMs（一个扮演用户，一个扮演助理）进行自我模拟对话，这些对话模拟了真实的用户与对话代理的多轮交互。
- 对话过程中，PPDPP 预测下一步的对话策略，根据交互历史将预测的动作映射到自然语言指令中。
- 通过这种方式，模拟出的对话数据用于强化学习过程。

### 4. 强化学习 (Reinforcement Learning)
- 在自我模拟过程中，一个第三方LLM作为奖励模型（reward model）提供目标导向的反馈。
- 这个奖励模型的输出被转换为标量奖励，用于强化学习过程。
- 当对话达到一个终点状态（目标完成或达到最大轮数），使用强化学习算法进一步调整PPDPP，以优化对话策略。

### 5. 评估和迭代
- 提出了一种基于LLM的交互性评估方法，使用LLM构建的用户模拟器和奖励模型来评估对话系统在实现指定目标方面的成功率和平均轮数。
- 通过这种评估方法，可以在多轮对话中测量对话系统的策略规划能力。

### 6. 应用到新情况
- PPDPP经过训练后，可以直接应用到多种新的对话情况，而无需再次进行多轮的模拟。

通过上述步骤，PPDPP方法旨在通过监督学习和强化学习提高LLM在主动对话中的策略规划能力，并在实验中展示了其在不同对话场景下的有效性和适用性。这种方法的关键在于它可以自适应地学习如何在多轮对话中实现特定的目标，并且可以通过替换插件来适应不同的应用场景。

>>>>>>
``` 

有用的信息: 

  - PPDPP的主要创新是使用LLM模拟对话(以生成训练数据), 以及用LLM作为评分器
  - 其中对于 "议价" 场景, 整理了常见对话策略: 
    - ![image2024-3-3 16:36:51.png](/assets/01KJBZ737JH1P7DJDHR56B8N4T/image2024-3-3%2016%3A36%3A51.png)
    - 类似于我们整理的DBA技术问题的讨论策略

# 论文: Adaplanner: Adaptive planning from feedback with language models

  

问答: 

```
在这篇论文中, 主要解决的问题是什么, 主要的逻辑线是什么

这篇论文的主要问题是如何改进大型语言模型（LLMs）在顺序决策任务中作为自主代理的表现。目前，大多数使用LLMs的方法要么是在没有计划的情况下贪婪地采取行动，要么是依赖于静态计划，这些计划不能适应环境反馈。这限制了LLMs在复杂问题和长期规划中的性能。

主要的逻辑线如下：

1. **问题介绍和动机**：文中首先指出了现有方法的局限性，即它们不能根据环境反馈进行动态调整，导致在面对问题复杂性增加和计划视野扩大时性能下降。

2. **AdaPlanner的提出**：为了解决这一问题，提出了一个封闭循环方法AdaPlanner，它使得LLM代理能够根据环境反馈适应性地修正自己生成的计划。

3. **方法细节**：
    - **计划生成和修正策略**：AdaPlanner利用代码风格的LLM提示结构来生成计划，并在执行过程中通过两种反馈（in-plan和out-of-plan）来适应性地修正计划。
    - **技能发现机制**：提出了一种技能发现机制，通过将成功的计划作为少量样本示例，使代理能够在更少的任务演示中进行规划和调整。

4. **实验验证**：在ALFWorld和MiniWoB++环境中进行实验，展示了AdaPlanner在使用更少样本的情况下，比现有的最先进方法性能提升了3.73%和4.11%。

5. **结论和讨论**：得出结论AdaPlanner通过封闭循环框架有效提升了规划性能，即使面对有限的样本。同时指出AdaPlanner的局限性，并讨论了未来可能的发展方向。

6. **更广泛的影响**：论文还讨论了LLMs作为自主代理在顺序决策任务中应用可能带来的广泛影响和潜在风险，并提出了研究者和政策制定者应共同努力，指导这些技术的发展和部署。

整篇论文围绕如何改进LLMs在顺序决策任务中的表现这一核心问题展开，提出了一个创新的方法，并通过实验验证了其有效性。

>>>>>>

在本论文中, 详细描述 "现有方法的局限性"

本论文中提到的现有方法的局限性主要包括以下几点：

1. **缺乏计划**：一些方法在没有预先规划的情况下贪婪地采取行动，这意味着它们只根据当前情况选择行动，而没有考虑未来的后果。这种方法可能在简单任务中有效，但在需要多步骤规划的复杂任务中表现不佳。

2. **静态计划**：其他方法依赖于预先生成的静态计划来指导行动。虽然这些方法比贪婪方法考虑得更长远，但它们在执行过程中不能根据环境反馈进行调整，导致它们无法适应计划中未预见到的变化。

3. **适应性差**：由于上述方法不能根据环境反馈进行动态调整，它们在面对问题复杂性增加和计划视野扩大时性能下降。这限制了它们在复杂环境中的应用。

4. **计划固定**：大部分现有的闭环方法生成一个固定的计划，并且仅在接收到环境反馈时更新它们的行动。这种方式可能会导致在短期内对环境做出适应性调整，但可能对未来步骤产生不利影响。

5. **数据需求量大**：例如，DEPS方法需要训练一个计划选择器来选择最可能成功的计划，这需要大量特定于任务的数据。这不仅增加了数据获取的成本，而且限制了方法的普遍适用性。

6. **泛化能力不足**：许多现有方法需要大量特定于任务的训练数据，缺乏在不同任务和环境之间泛化的能力。

总的来说，现有方法要么无法在计划执行过程中适应环境变化，要么在数据需求和泛化能力上存在限制。这些局限性导致了LLM代理在顺序决策任务中的性能不足，特别是在面对复杂任务或长期规划时。论文提出的AdaPlanner方法旨在通过引入自适应闭环规划来克服这些限制。

>>>>>>

在本论文中, 描述AdaPlanner的方法细节

在论文中，AdaPlanner的方法细节主要围绕以下几个核心组件：

### 3.1 计划生成 (Plan Generation via Code-Based LLM Prompting)

AdaPlanner使用基于代码的提示来指导LLMs生成计划。与自然语言提示相比，代码提示能够减少模糊性和误解，显著降低LLMs在计划生成和调整过程中出现的幻觉（hallucination）现象。具体来说，AdaPlanner在不同的决策阶段设计了针对性的代码提示，包括自适应规划、反馈生成和在任务中的调整。

### 3.2 自适应闭环计划调整 (Adaptive Closed-Loop Plan Refinement)

AdaPlanner在生成初步计划后，通过执行环境接口，使得计划中的动作能够在环境中得到体现，并且能够根据环境返回的观察结果进行计划的调整。该方法区分了两种环境反馈：

- **在计划内反馈 (In-Plan Feedback)**：如果环境反馈与预测一致，AdaPlanner利用特定的`ask_LLM()`动作提取反馈中有用的信息，用于即将到来的动作。
- **计划外反馈 (Out-of-Plan Feedback)**：如果环境反馈与预测不一致，AdaPlanner将修正整个计划，并确定从哪个中间点继续执行任务。

### 3.3 技能发现 (Skill Discovery)

AdaPlanner实现了一种技能发现机制，通过存储成功的计划并将其作为未来规划的指导，提高了抽样效率和可靠性。这个过程包括两个阶段：技能获取（Skill Acquisition）和技能过滤（Skill Filtering）。在技能获取阶段，AdaPlanner探索并细化解决方案，成功完成任务后，将最新的解决方案视为候选技能。在技能过滤阶段，将这些解决方案集成到提示中以提升成功率，那些能够提升性能的解决方案会被存档为发现的技能，否则会被丢弃。

### 实验验证 (Evaluation)

AdaPlanner在两个文本决策环境ALFWorld和MiniWoB++中进行了测试。实验结果显示，AdaPlanner在使用更少样本的情况下，比现有最先进方法的成功率分别提高了3.73%和4.11%。

### 结论 (Conclusion)

AdaPlanner通过引入自适应闭环规划框架，显著提升了LLM代理的顺序决策性能。研究表明，即使在样本数量有限的情况下，AdaPlanner也能有效利用环境反馈来增强其规划能力。

### 局限性 (Limitations)

AdaPlanner虽然提高了样本效率，但在处理复杂任务时仍需要一些专家演示。未来的研究可探索如何进一步增强AdaPlanner在没有演示的情况下解决复杂任务的能力。

### 更广泛的影响 (Broader Impacts)

研究者们强调需要共同努力制定有效的监管措施，以指导LLM代理的发展和部署，确保它们的应用能够产生积极的社会影响。

>>>>>>

在这篇论文中, AdaPlanner的局限性是什么

在论文中提到的AdaPlanner的局限性主要包括以下几点：

1. **需要少量专家演示**：尽管AdaPlanner提高了样本效率，但它仍然依赖于少量的专家演示来解决复杂的任务。这意味着在没有任何演示或任务指导的情况下，AdaPlanner可能无法有效工作。

2. **泛化性的潜在问题**：AdaPlanner的技能发现机制依赖于存储成功的计划来辅助未来的任务。这可能会导致模型依赖于特定的成功案例，而不是学习更通用的解决策略。

3. **环境交互的计算成本**：AdaPlanner在执行过程中需要与环境进行交互来获取反馈并据此调整计划。这种交互可能会带来额外的计算开销，特别是在需要频繁调整计划的复杂环境中。

4. **执行效率**：虽然AdaPlanner使用闭环控制系统能够进行实时反馈和计划调整，但这可能导致执行效率下降，特别是在计划需要频繁更新或环境反馈较为复杂时。

5. **代码提示的局限性**：AdaPlanner利用代码风格的提示来减少LLMs的幻觉问题，但这种方法可能不适用于所有类型的顺序决策任务，特别是在那些难以用代码风格来表达的任务中。

6. **对模型质量的依赖**：尽管AdaPlanner设计用来减少对模型的依赖，但它在某种程度上仍依赖于底层LLM的质量，如模型的理解能力和生成的行动的相关性。

7. **现实世界应用的挑战**：尽管在仿真环境中获得了良好的性能，AdaPlanner在现实世界应用时可能面临额外的挑战，如环境噪声、未知因素和动态变化等。

论文的作者们意识到这些局限性，并建议未来的研究可能会探讨如何进一步提升AdaPlanner在没有现有演示的情况下解决复杂任务的能力，以及如何将其应用于更广泛的现实世界问题中。

>>>>>>

在这篇论文中, Closed-Loop Plan 与 open-loop有什么区别, 为什么要做这样的区分

在论文中，“Closed-Loop Plan”和“Open-Loop”表示两种不同的决策策略，它们在如何处理环境反馈方面有本质的不同。以下是两者的区别以及进行这样区分的原因：

### Open-Loop Methods:

- **静态计划**：Open-loop 方法依赖于预先确定的计划，这些计划在执行过程中不会根据环境反馈进行调整。
- **计算开销小**：由于计划是静态的，因此它们在计算上更简单、更快。
- **适应性差**：这些系统不会监控环境的变化，因此一旦环境发生变化或计划执行过程中出现意外情况，这些方法可能无法有效响应。
- **适用性限制**：Open-loop 方法适用于预测性较强、变化不大的环境，但在动态或不确定性较高的环境中性能可能会下降。

### Closed-Loop Systems:

- **动态调整**：Closed-loop 系统会将环境反馈整合到决策过程中，根据反馈调整未来的计划和行动。
- **适应性强**：这样的系统能够对环境变化做出响应，调整初始计划以应对实际情况的偏差。
- **计算开销大**：与Open-loop方法相比，Closed-loop系统可能需要更多的计算资源，因为它们需要不断评估环境反馈并重新规划。
- **更适合复杂环境**：Closed-loop方法更适合解决复杂、多变或需要长期规划的任务，因为它们能够利用即时反馈来优化行动策略。

### 为什么要做这样的区分？

- **性能差异**：在顺序决策任务中，一个主要的挑战是如何根据环境反馈适应性地做出决策。Open-loop 和 Closed-loop 方法在这方面存在明显的性能差异。
- **策略选择**：理解两者的区别有助于研究人员根据不同任务的特点和需求选择或设计合适的决策策略。
- **改进动机**：通过明确现有方法的缺点，研究人员可以更好地设计新的策略（如AdaPlanner）来克服这些限制，提高在复杂环境中的任务性能。

AdaPlanner采用了Closed-loop 系统的策略，并通过自适应地调整计划来增强LLM代理在顺序决策任务中的性能。这样的设计使其能够更好地面对环境的不确定性和复杂性，从而在实验中取得了比Open-loop和其他Closed-loop方法更好的性能。

>>>>>>

在这篇文章中, 翻译这一段文字: 

When AdaPlanner observes that the
environment is aligned with the anticipated plan, it performs in-plan refinement. This allows it to
extract useful information from the observation that can be used for upcoming actions. To achieve
this, we provide the agent with an additional action called ask_LLM(), which is used to formulate
a plan alongside task-specific actions. The ask_LLM() function enables AdaPlanner to self-query
and perform reasoning based on specific information parsed from environmental observations. For
instance, in [Step 3] in Figure 2 (a), the ask_LLM() action extracts the identifier of the found object lettuce from the natural-language observation. This information can then be fed into later
actions. As an additional atomic action, this in-plan refinement is integrated into the plan at any
point where the planner deems a reasoning process is necessary. Existing code-generation-based
methods [19, 10, 3] face a challenge in this task, especially when there is no prior knowledge of the
structure and organization of these feedback sentences. In contrast, our AdaPlanner method leverages
LLM to parse critical information from diverse feedback presented in natural-language sentences to
streamline plan execution

当AdaPlanner观察到环境与预期的计划一致时，它会执行计划内的细化。这使它能够从观察中提取有用的信息，用于即将到来的行动。为了实现这一点，我们为代理提供了一个额外的动作，称为ask_LLM()，它用于制定计划以及特定任务的行动。ask_LLM()函数使AdaPlanner能够自我查询并基于从环境观察中解析的特定信息进行推理。例如，在图2(a)的[步骤3]中，ask_LLM()动作从自然语言观察中提取了找到的对象生菜的标识符。然后，这些信息可以输入到后续的行动中。作为一个额外的原子行动，这种计划内的细化在规划者认为需要推理过程的任何点都被集成到计划中。现有的基于代码生成的方法[19, 10, 3]在这个任务中面临挑战，特别是当没有先前对这些反馈句子的结构和组织的知识时。与之相反，我们的AdaPlanner方法利用LLM从多样化的自然语言句子中提取关键信息，以简化计划执行。

>>>>>>

在这篇论文中, 翻译以下文字:

After each sub-plan execution, AdaPlanner actively checks an assertion condition to ensure that the current plan is proceeding
as expected. If the assertion fails, AdaPlanner performs out-of-plan refinement. For example, in
Figure 2 (a), after [Step 3], the agent is expected to hold lettuce. If this condition is not met,
AdaPlanner generates an error message that details the current progress of execution gathered by the
report() function. In ALFWorld tasks, this function provides a report of the agent’s location, the
object it is holding, and the last three interactions with the environment, as shown in Figure 2 (b).
AdaPlanner then utilizes this information to perform out-of-plan refinement.
During the out-of-plan refinement as in Figure 2 (c), AdaPlanner uses a prompt similar to the one
used during the initial planning stage, but with an additional feedback message that reflects the
current state. Detailed prompts are provided in Appendix 8.3. AdaPlanner then refines the plan based
on the newly acquired information and also determines the value of start_from by comparing
the plan before and after the refinement. The newly refined solution() is then executed from
the breakpoint start_from. This breakpoint contains all variable states that were saved prior to
refinement. Consequently, the current episode can continue from an intermediate checkpoint without
restarting from scratch. We call this mechanism refine-then-resume. It significantly speeds up task
completion and reduces the number of LLM calls required.

在每个子计划执行后，AdaPlanner会积极检查一个断言条件以确保当前计划按预期进行。如果断言失败，AdaPlanner会执行计划外的细化。例如，在图2(a)的[步骤3]之后，预期代理会持有生菜。如果没有满足这个条件，AdaPlanner会生成一个错误信息，该信息详细描述了由report()函数收集的当前执行进度。在ALFWorld任务中，此函数提供代理的位置、它持有的对象以及与环境的最后三次互动的报告，如图2(b)所示。然后AdaPlanner利用这些信息执行计划外的细化。

在计划外细化过程中，如图2(c)所示，AdaPlanner使用类似于初始规划阶段使用的提示，但添加了反映当前状态的额外反馈信息。详细的提示在附录8.3中提供。AdaPlanner然后根据新获得的信息细化计划，并通过比较细化前后的计划确定start_from的值。新细化的solution()函数随后从断点start_from开始执行。这个断点包含了细化前保存的所有变量状态。因此，当前剧集可以从一个中间检查点继续进行，而无需从头开始。我们称这种机制为细化后继续。它显著加快了任务完成的速度，并减少了所需的LLM调用次数。

>>>>>>

在这篇论文中, 什么是  refine-then-resume 机制

在论文中，"refine-then-resume" 机制是指AdaPlanner在执行计划时遇到与预期不符的情况后，不是从头开始整个任务，而是对当前执行的计划进行调整，并从中断的地方继续执行任务的过程。

具体来说，这个机制包含以下几个步骤：

1. **执行子计划**：AdaPlanner按照生成的计划逐步执行任务。
2. **断言检查**：在每个子计划执行后，AdaPlanner会检查一个断言条件，以确认任务是否按照计划进行。
3. **识别问题和计划外细化**：如果断言检查失败，表明任务执行与计划不符，AdaPlanner会生成一个错误报告，并根据报告中的信息对计划进行外部细化。
4. **确定恢复点**：AdaPlanner通过比较计划外细化前后的计划来确定从哪一步开始恢复执行(`start_from`)。
5. **恢复执行**：一旦计划被修正，AdaPlanner会从确定的恢复点（`start_from`）重新开始执行任务，而不是从最开始重新执行整个任务。

这个机制的优点在于显著提高了任务完成的效率，因为它避免了在每次遇到执行问题时都需要从头开始，而是允许代理从中断点继续执行，这样可以节约大量的时间和计算资源。此外，它还减少了与LLM的交互次数，因为不需要每次都重新生成整个计划。

>>>>>>

在这篇论文中, 什么是 Skill Discovery

在这篇论文中，"Skill Discovery" 是指AdaPlanner中的一种机制，它允许模型从成功执行的任务中学习，并将这些经验用作未来任务规划的参考。这个过程帮助模型在面对类似的任务时，能够更高效和有效地生成行动计划。

Skill Discovery机制包括两个主要阶段：

1. **技能获取 (Skill Acquisition)**：在此阶段，AdaPlanner尝试解决新的、未见过的任务，利用有限数量的人类示例或甚至没有示例来指导它。它通过自适应闭环规划的迭代探索和细化过程，尝试并错误地找到解决方案。一旦AdaPlanner成功完成任务，它将最新的解决方案和相应的环境交互视为候选的发现技能。

2. **技能过滤 (Skill Filtering)**：在此阶段，会评估所发现的解决方案能否提高任务的成功率。将解决方案整合到提示中，如果它们能够提升性能，则将这些解决方案存档为"发现的技能"。如果它们不能提升性能，则会被舍弃。这一过程对于确保通过迭代闭环细化集成到修订解决方案中的特定于情境的信息不会妨碍其广泛适用性至关重要。

通过Skill Discovery机制，AdaPlanner可以不断丰富其"技能库"，提高在处理相似任务时的规划能力和样本效率，从而在没有大量示例的情况下也能成功解决复杂任务。这有助于减少对昂贵和难以获取的专家轨迹的依赖。

>>>>>>

``` 

  

有用的信息: 

  - 给大模型的提示词, 使用代码会更精确 (让大模型生成代码, 也会更精确)
    - 可以让大模型来检查代码的Syntax error
    - 在大模型对抗中, 也可以使用代码来描述计划步骤
  - 当反馈系统认为当前计划错误时, 进入Closed-loop. 
    - 分析原因并修复计划 (这部分并没有创新信息, 仅创建了一个词Closed-loop)
  - Skill Discovery 就是基本的反馈学习系统  

# 信息整理

```
对解决方案有用的信息: 
- 通过提示词, 来整理一个 Topic 的堆栈. (前提假设: 对于诊断类的对话, 其核心是堆栈类的问答, 以及一些跳转)
- 让系统和用户模拟器(LLM)对抗, 产生训练样本
- 用LLM做出评分器
- 规范LLM的对话策略
- DEPS步骤, 作为LLM的思考步骤

对提示词有用的信息: 
- 在提示词中, 指明 Topic 的受众是AI (前提假设: 给AI的Topic描述, 比给人类的描述 会更精确? )
- 给大模型的提示词, 使用代码会更精确. 在大模型对抗中, 也可以使用代码来描述计划步骤
```
