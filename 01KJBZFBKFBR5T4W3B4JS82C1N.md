---
title: 20240910 - 对提示词自动调优(ai prompter)的开发评估
confluence_page_id: 3342360
created_at: 2024-09-10T06:51:36+00:00
updated_at: 2024-09-12T09:30:50+00:00
---

由张乾实现了一个简单的版本 (参考dspy和autoprompt): 

[refine_autoprompt-0910.ipynb](/assets/01KJBZFBKFBR5T4W3B4JS82C1N/refine_autoprompt-0910.ipynb)

# 流程分析

  1. 样本数据: ChatDBA 的 refine阶段的输入, 包括对话历史, 参考资料, 和回答框架. 但不包括指令
  2. 进行 num_steps 轮迭代: 
     1. 对样例数据, 进行答案推理 (generation_prompt_template) 
     2. 对生成的答案, 进行评分评估(eval_prompt_template)
     3. 差错分析 (error_analysis_prompt_template)
     4. 检查终止条件: stop_criteria
     5. 根据差错分析, 生成新的提示词: new_generation_prompt_template
     6. 如果样例数据不足: 
        1. 生成挑战样例: challenge_samples_prompt_template

# 需要解决的问题

总的目标: 复现ChatDBA 1.0的人工调优的结果

55 incomplete 单一样例下, 是否能过拟合 (评估优化梯度是否健康) 61 incomplete 等待将新的评估指标融合进代码, 再检查过拟合是否能形成 62 incomplete 梯度效率 56 incomplete 挑战样例对梯度到底是否有影响 (是否精简架构) 58 incomplete 多维度标准 + 多样例下, 是否能正确优化 59 incomplete 引入更多优化手法 (交换章节, 标记局部重点, 方向性指导), 获得更优的优化梯度 60 incomplete 估价函数: 评分系统 v.s. 负向标记系统, 需要符合DBA的使用习惯 (使用标记平台 + AI标记学习) 

低优先级: 

57 incomplete 由于没有 给样例 给定 Ground Truth, 那当前评分系统是否符合DBA的判断 

# 先改进现在的评分方式, 让评分拉开, 才能评估过拟合 以及 挑战样例 的效果

更换一组更容易区别的指标, 用代码测试分离度, 简单抽样下分离度还可以

[refine_autoprompt-0910-Copy1.html](/assets/01KJBZFBKFBR5T4W3B4JS82C1N/refine_autoprompt-0910-Copy1.html)

![image2024-9-11 15:47:36.png](/assets/01KJBZFBKFBR5T4W3B4JS82C1N/image2024-9-11%2015%3A47%3A36.png)

评估挑战用例时, 分值起点会不太稳定. 需要改善评分提示词, 并使用多次评分的平均值来降低方差. 

更新后的提示词: [refine_autoprompt-0910-Copy1 (1).html](/assets/01KJBZFBKFBR5T4W3B4JS82C1N/refine_autoprompt-0910-Copy1%20%281%29.html)

使用/不使用 挑战用例时, 梯度的差异对比: 

![image2024-9-12 13:48:26.png](/assets/01KJBZFBKFBR5T4W3B4JS82C1N/image2024-9-12%2013%3A48%3A26.png)

纵向是用例, 横向是轮次, -1是评分输出格式错误导致无法解析

(

红色的这个挑战用例是一直没调优成功的.   
蓝色的那个用例, 是挑战成功, 但后续调优成功了.

)

看上去用例挑战性不强, 对梯度的提升效果也看不出来: 

  - (带挑战用例)
    - ![image2024-9-12 13:55:19.png](/assets/01KJBZFBKFBR5T4W3B4JS82C1N/image2024-9-12%2013%3A55%3A19.png)
  - (不带挑战用例)
    - ![image2024-9-12 13:55:46.png](/assets/01KJBZFBKFBR5T4W3B4JS82C1N/image2024-9-12%2013%3A55%3A46.png)
  - 需要分析挑战用例的细节

# 尝试: 负向标记系统 转化成 评分模型

先假设 负向标记数据 是存在的, 推导后面的流程 (差错分析 + 新提示词生成) 是否通畅并有效. 

带标记的差错分析提示词: 

```
Assistant is a large language model designed to provide a high quality analysis for every task.
You are given the following task description
你是一个DBA专家, 需要回答用户关于数据库的技术问题
 
Here is the prompt instructions that was given to the model:
你是一个DBA专家, 需要逐步分析, 然后提供你的判断, 最后提供应急手段
 
An expert ranker evaluated the model's performance on the given task description.
and rank according to the following scale: 可解释性, 引导性
 
The mean score for this prompt is: 4.0
##
Here is a list of challenging cases for the given prompt and their rank and ranker's comment:
##Challenging Cases:
Case: {'question': '如何进行MySQL备份', 'answer_with_ranker_comment': '使用mysqldump工具<<ranker's comment: 引导性: 此处应当提供样例命令>>', rank: 1.0}
Case: {'question': '如何进行MySQL备份', 'answer_with_ranker_comment': '使用xtrabackup工具, 详细的命令是: .... <<ranker's comment: 可解释性: 应当给出命令中常用参数的解释>>', rank: 2.0}
Case: {'question':  '请解释一下Python中的列表推导式', 'answer_with_ranker_comment': '列表推导式是一种快速创建列表的方法。<<ranker's comment: 缺少举例，建议补充至少一个代码示例>>', rank: 0.8}
Case: {'question': 'Redis有哪些数据类型？', 'answer_with_ranker_comment': 'Redis有字符串、列表、哈希、集合。<<ranker's comment: 事实错误，Redis还有有序集合和位图等数据类型>>', rank: 0.5}
Case: {'question': '如何学习机器学习？', 'answer_with_ranker_comment': '去看书吧。<<ranker's comment: 引导性不足，可以提供一些书籍推荐或者学习路径>>', rank: 0.6}
Case: {'question': 'Vue.js有什么优点？', 'answer_with_ranker_comment': 'Vue.js很简单，谁都会用。<<ranker's comment: 语气不佳，建议用更专业的表述，例如“易于学习和使用”>>', rank: 0.7}
Case: {'question': '什么是RESTful API？', 'answer_with_ranker_comment': 'RESTful API是一种设计风格。<<ranker's comment: 过于简略，建议解释RESTful API的设计理念和约束条件>>', rank: 0.6}
Case: {'question': 'Java和Python哪个更好？', 'answer_with_ranker_comment': '当然是Java好！<<ranker's comment:  缺乏客观性，应该说明两者各有优缺点，适用于不同的场景>>', rank: 0.4}
Case: {'question': '怎么用CSS实现一个圆形按钮？', 'answer_with_ranker_comment': '设置border-radius: 50%。<<ranker's comment:  缺少上下文，建议补充完整代码示例，包括HTML和CSS部分>>', rank: 0.7}
Case: {'question': 'Docker有什么用？', 'answer_with_ranker_comment': 'Docker可以用来做容器化。<<ranker's comment:  过于抽象，建议具体说明Docker的应用场景和优势>>', rank: 0.6}
Case: {'question': 'Git和SVN有什么区别？', 'answer_with_ranker_comment': 'Git是分布式的，SVN是集中式的。<<ranker's comment:  过于简略，建议从多个维度对比两者的差异，例如分支管理、存储方式等>>', rank: 0.7}
Case: {'question': '如何优化网站SEO？', 'answer_with_ranker_comment': '做关键词排名。<<ranker's comment:  过于片面，SEO是一个系统工程，建议提供更全面的优化建议>>', rank: 0.5}

###
Note that the ranker labels are __absolutely correct__, but the prompts (task descriptions) may be incorrect and need modification.
Your task is to provide a brief analysis of the given prompt performance.
Guidelines:
1. The analysis should contain only the following information:
    - A summary of the common mistakes of the prompt and the ways he can be improve his generation, try to cluster the failure cases into groups and describe each group.
    - You should read ranker's comment carefully
2. The total length of your analysis should be less than 200 token!
Analysis(in Chinese):
``` 

输出: 

```
模型在可解释性和引导性方面表现不佳，回答过于简略，缺乏深度和细节。主要问题可以归纳为以下几类：

**1. 缺乏细节和示例：** 对于操作步骤类问题，仅提供工具或方法，缺乏具体的命令、参数解释和代码示例，例如案例 1、2、3、9。改进方法：提供详细的操作步骤、代码示例，并对关键参数进行解释。

**2. 过于笼统，缺乏深度：** 对于概念解释类问题，仅提供简单的定义，缺乏对概念的深入解读、应用场景和优缺点分析，例如案例 7、10、11、12。改进方法：补充背景知识、应用场景、优缺点对比等，进行更全面的解释。

**3. 语气和客观性：** 部分回答语气过于随意，缺乏专业性，例如案例 6。 同时，在比较不同技术时，缺乏客观性，例如案例 8。改进方法：使用更专业的表达，避免主观判断，客观分析不同技术的优缺点和适用场景。
``` 

从结果上看, 差错分析的总结 结果正确

新提示词生成: 

```
Assistant is a large language model designed to provide the best instructions for every task.
Below are a few suggested instructions for the task and score (mean of the rank), for the following task description:
你是一个DBA专家, 需要回答用户关于数据库的技术问题
 
## Examples
你是一个DBA专家, 需要逐步分析, 然后提供你的判断, 最后提供应急手段
######
This is the analysis for the last instruction:
模型在可解释性和引导性方面表现不佳，回答过于简略，缺乏深度和细节。主要问题可以归纳为以下几类：

**1. 缺乏细节和示例：** 对于操作步骤类问题，仅提供工具或方法，缺乏具体的命令、参数解释和代码示例，例如案例 1、2、3、9。改进方法：提供详细的操作步骤、代码示例，并对关键参数进行解释。

**2. 过于笼统，缺乏深度：** 对于概念解释类问题，仅提供简单的定义，缺乏对概念的深入解读、应用场景和优缺点分析，例如案例 7、10、11、12。改进方法：补充背景知识、应用场景、优缺点对比等，进行更全面的解释。

**3. 语气和客观性：** 部分回答语气过于随意，缺乏专业性，例如案例 6。 同时，在比较不同技术时，缺乏客观性，例如案例 8。改进方法：使用更专业的表达，避免主观判断，客观分析不同技术的优缺点和适用场景。
######
Your task is to generate:
1. A new instruction that is
    -Different from all the instructions above
    -Follows exactly the error analysis modification suggestions, and fix the instruction to improve the quality of the instruction.
    -Has a higher score than all the instructions above.
2. The predicted score of this instructions
 
You must adhere the error analysis instructions! even in case it seems there is a contradiction between these instructions, and the task. The error analysis was evaluate by an expert ranker, thus represent the exact intent of the task.
The generated instruction should be phrased as a clear generation instruction! it should not include any instructions and descriptions on the modification that should be done to the instruction.
Note that the previous instruction contains an implicit assumptions on the intent of the task that might be incorrect. You should replace this assumption with more accurate assumptions using the score of the previous instructions and the error analysis.
Prompt:
``` 

结果: 

```
## New Instruction:

你是一位经验丰富的数据库管理员（DBA）专家，需要为用户解答关于数据库的技术问题。用户的问题可能涉及数据库的各种概念、操作、故障排除等方面。请你扮演 DBA 专家的角色，以专业、严谨的态度为用户提供详细、深入、易懂的解答。

在回答用户问题时，请务必做到以下几点：

* **详细分析问题：** 仔细阅读用户的问题，明确用户的具体需求和问题背景。
* **逐步解释概念：**  对于涉及数据库概念的问题，不仅要给出定义，还要解释其背后的原理、应用场景、优缺点等，帮助用户深入理解。
* **提供具体操作步骤：**  对于涉及数据库操作的问题，要提供详细的操作步骤、代码示例，并对关键参数进行解释，确保用户能够按照步骤顺利完成操作。
* **结合实际案例：**  尽可能结合实际案例来解释概念或说明操作步骤，增强答案的实用性和可理解性。
* **客观比较不同技术：**  在比较不同数据库技术时，要保持客观公正，避免主观判断，从多个维度分析其优缺点和适用场景。
* **使用专业术语：**  使用准确、专业的数据库术语，但要避免过度使用专业术语，确保用户能够理解你的回答。

请记住，你的目标是为用户提供高质量、有价值的数据库技术支持，帮助他们解决问题、提升技能。

## Predicted Score: 

This instruction is predicted to receive a score of **4.5** or higher. 

**Reasoning:**

The original instructions and their low scores reveal a crucial misunderstanding of the task's intent. The task is not simply to provide short answers as a chatbot might. Instead, the high emphasis on "step-by-step analysis," "detailed explanations," and "practical examples" suggests the user desires comprehensive and instructive responses akin to a patient teacher or a technical guide. 

This new instruction addresses the identified weaknesses by explicitly mandating:

* **Thoroughness:**  Moving beyond simple definitions to encompass principles, applications, advantages, and disadvantages.
* **Practicality:**  Emphasizing detailed steps, code samples, parameter explanations, and real-world examples for actionable guidance.
* **Objectivity:**  Promoting unbiased comparisons of technologies based on various factors rather than personal preferences.
* **Clarity:**  Balancing the use of precise terminology with accessible language for a wider audience.

By incorporating these elements, the new instruction guides the model to generate responses that are not only accurate but also deeply informative, practically useful, and engaging for users seeking to enhance their understanding of databases.
``` 

目前认为负向标记可以帮助后续流程进行改进, 指导改进方向.

存在的问题是: 

  1. 如果负向标记的问题种类过多, 那么在差错分析中, 差错样例的选择就比较重要
  2. 对于多个问题方向, 目前提示词改进的方式是 列举, 但对于单个方向的改进深度可能太浅, 或者方法不足
     1. 改进举例: 通过跟ranker用户的对话, 将标记进行扩展, 获得更准确的方向
     2. 改进举例: 让ranker写出预期的答案, 与现有答案进行对比, 获取更准确的深度

下一步: 用Llama-factory, 尝试sft+ppo, 训练一个用于负向标记的模型
