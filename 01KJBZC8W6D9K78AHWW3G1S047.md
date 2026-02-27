---
title: 20240730 - 理解Dspy
confluence_page_id: 3145901
created_at: 2024-07-30T02:33:46+00:00
updated_at: 2024-08-02T11:53:23+00:00
---

# 初步代码

[dspy.html](/assets/01KJBZC8W6D9K78AHWW3G1S047/dspy.html)

# 概念

  - Signature, 对LLM调用的函数签名
    - 签名需要单独指定的原因: 提示词中会使用"输入"和"输出", 与签名的参数是对应的. 这样在调优提示词的过程中, 就需要签名的参数来生成 新的提示词
  - Predictor, 对LLM调用的函数

# 优化器代码目录: 

<https://github.com/stanfordnlp/dspy/blob/b1da1af7b87f52de56c790dc2a407b188d70ea1a/dspy/teleprompt/copro_optimizer.py>

# 对优化器BootstrapFewShot进行解析

<https://github.com/stanfordnlp/dspy/blob/b1da1af7b87f52de56c790dc2a407b188d70ea1a/dspy/teleprompt/bootstrap.py#L35>

  - 配置teacher模型  

    - 如果不指定teacher模型, 则使用student模型作为teacher
    - 如果指定了max_labeled_demos, 则对teacher模型使用 LabeledFewShot(k=max_labeled_demos) 来进行优化 (compiled)
  - "训练"过程 (不是真的训练)
    - 对于trainset中的输入, 使用teacher模型进行回答, 并评分. 取评分超过阈值的 作为 增强后的demo
    - 取max_bootstrapped_demos个 增强后的demo, 拼接校验集中随机的demo, 总共拼max_labeled_demos个, 拼接到提示词中, 作为few-shot的例子

对优化器结构的理解: 

  - BootstrapFewShot, 相当于训练器, 指定了训练的方法
  - 训练器中的teacher/student, 是模块(Module), 相当于提示词方法论, 比如ChainOfThought
    - 模块可以指定多个步骤的组合, 比如: 
    - (RAG) ![image2024-7-30 15:7:47.png](/assets/01KJBZC8W6D9K78AHWW3G1S047/image2024-7-30%2015%3A7%3A47.png)
    - (多跳QA) ![image2024-7-30 15:14:8.png](/assets/01KJBZC8W6D9K78AHWW3G1S047/image2024-7-30%2015%3A14%3A8.png)
  - 模块中, 会调用Predict, 相当于对大模型的一次调用. 
  - Predict是通过Signature生成的, Signature相当于LLM调用的函数签名

# 对优化器BootstrapFewShotWithRandomSearch进行解析

评估使用的teacher模型, 是用max_labeled_demos个few-shot, 进行LabeledFewShot优化

每个评估, 都使用teacher模型, 根据输入生成标准输出

一共进行几轮优化 (-3轮, -2轮, -1轮, 0..num_candidate_sets-3 轮), 选择效果最好的结果: 

  - -3轮: 直接使用zero shot来评估
  - -2轮: 使用LabeledFewShot进行优化后, 评估
  - -1轮: 将所有训练数据, 传给BootstrapFewShot, 评估
  - 0..num_candidate_sets-3 轮: 每轮选择训练数据的一个子集 (数量介于 min_num_samples 和 max_num_samples), 将子集传给BootstrapFewShot, 评估

# 对优化器LabeledFewShot进行解析

直接将训练数据 (数据+标注), 直接写在few-shot中

# 对优化器Ensemble进行解析

将多个优化器的结果进行聚合

# 对优化器BootstrapFinetune进行解析

  - 用teacher和训练数据, 对student进行优化
  - 用优化后的student, 生成微调数据
  - 用微调数据 对 target指定的模型进行微调
  - 整个过程, 相当于进行模型蒸馏

# 对优化器COPRO进行解析

  - "COPRO" 是 Combinatorial Prompt Ranking Optimization 的缩写
  - 调优一共进行depth轮
    - 每轮: 
      - 使用 之前发现的评分最高的提示词 作为few-shot, 生成新的提示词候选
      - 对新的提示词候选进行评分, 进入下一轮
    - 选取评分最高的提示词
  - 问题: COPRO用最优解生成新的最优解, 但由于LLM的想象力有限, 容易很快收敛, 停在局部解
  - 详细过程, 参考 [20240801 - dspy 实验](<http://8.134.54.170:8330/pages/viewpage.action?pageId=3145937>)

# 对优化器KNNFewShot进行解析

  - 通过KNN, 在训练集中进行样例召回
  - 用召回的样例, 构成few-shot

# 对优化器 BootstrapFewShotWithOptuna 进行解析

  - 优化器使用optuna算法 (超参数优化器)
    - 主要是想从一个大的样例集里, 挑出一组样例, 使得评分最好.
    - 而optuna优化后的"参数", 指的就是挑出的样例在样例集中的编号.
    - optuna适合从高维空间中进行采样优化. 
      - 对于一个复杂的chain, 每个环节相当于一个维度, 构成了高维空间

# 对优化器MIPROv2进行解析

  - "MIPRO" 是 Multiple Instruction, Prompt, and Ranking Optimization 的缩写，代表 多指令、提示和排名优化
  - 优化器使用optuna算法 (超参数优化器)
  - 与BootstrapFewShotWithOptuna的区别, 是MIPROv2 的优化方向是调优指令, 并不是只选择样例
  - 调优指令的生成, 使用了GroundedProposer
    - GroundedProposer初始化时, 会对数据集进行概述: 
      - <https://github.com/stanfordnlp/dspy/blob/4ad18d2a8a259d9366b667bb1c5afdc2f8459d47/dspy/propose/grounded_proposer.py#L260>
      - 提示词: DatasetDescriptorWithPriorObservations
    - 规定了几个调优方向: 
      - ![image2024-7-31 16:41:9.png](/assets/01KJBZC8W6D9K78AHWW3G1S047/image2024-7-31%2016%3A41%3A9.png)
    - 优化提示词的生成: 参考: GenerateSingleModuleInstruction
    - 生成这几个方向的指令, 通过Optuna来选择
  - 注意: Optuna的优势 是在高维空间/大面积空间中进行搜索. 

    - 如果predicator只有一个 (空间维度为1), 且 num_candidates 较小 (空间面积) 的情况下，Optuna 没有优势 

      - (num_batches (尝试次数) 大于 num_candidates时, 不如使用遍历扫描)

  - 注意: MIPROv2 只能对提示词进行一次调优, 而不能继续调优结果继续演进

  - 详细过程, 参考 [20240801 - dspy 实验]

# TODO

解释这段代码 (X上的宣传dspy的代码) 的用法: 

![image2024-7-30 15:31:5.png](/assets/01KJBZC8W6D9K78AHWW3G1S047/image2024-7-30%2015%3A31%3A5.png)

解释医药行业dspy应用的代码: <https://colab.research.google.com/drive/1CpsOiLiLYKeGrhmq579_FmtGsD5uZ3Qe#scrollTo=hVrLgbZvbJ97>

用断言来引导调优的方向:

  - <https://arxiv.org/pdf/2312.13382>
  - 代码: <https://colab.research.google.com/github/stanfordnlp/dspy/blob/main/examples/longformqa/longformqa_assertions.ipynb#scrollTo=Ysc2Gl8VimWY>
