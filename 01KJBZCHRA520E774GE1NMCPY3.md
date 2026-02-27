---
title: 20240801 - dspy 实验
confluence_page_id: 3145937
created_at: 2024-08-01T03:28:20+00:00
updated_at: 2024-08-02T11:50:36+00:00
---

# 参考

dspy cheetsheet: <https://dspy-docs.vercel.app/docs/cheatsheet>

# COPRO

[dspy-COPRO.html](/assets/01KJBZCHRA520E774GE1NMCPY3/dspy-COPRO.html)

[dspy-trail.html](/assets/01KJBZCHRA520E774GE1NMCPY3/dspy-trail.html)

过程解析: 

一共进行depth轮. 每轮开始: 

![image2024-8-2 19:47:49.png](/assets/01KJBZCHRA520E774GE1NMCPY3/image2024-8-2%2019%3A47%3A49.png)

会总结之前所有的提示词候选, 将评分最高的列出, 据此生成breath个 新的提示词候选:

![image2024-8-2 19:48:56.png](/assets/01KJBZCHRA520E774GE1NMCPY3/image2024-8-2%2019%3A48%3A56.png)

然后对于新增的提示词候选进行评分

![image2024-8-2 19:49:16.png](/assets/01KJBZCHRA520E774GE1NMCPY3/image2024-8-2%2019%3A49%3A16.png)

depth轮都结束后, 选择评分最高的提示词

进行Eval: 输出评分+结果列表

![image2024-8-1 11:28:3.png](/assets/01KJBZCHRA520E774GE1NMCPY3/image2024-8-1%2011%3A28%3A3.png)

# MIPROv2

[dspy-MIPROv2.html](/assets/01KJBZCHRA520E774GE1NMCPY3/dspy-MIPROv2.html)

重点解析: 

  - 生成对数据集的描述 (GroundedProposer的初始化过程)
    - ![image2024-8-1 16:59:27.png](/assets/01KJBZCHRA520E774GE1NMCPY3/image2024-8-1%2016%3A59%3A27.png)
  - 生成 num_candidates 个 few-shot 数据集
    - 多组数据集的生成方法, 与BootstrapFewShotWithRandomSearch的生成方法一样 (一共N个, 前三个有特殊含义)
    - ![image2024-8-1 20:1:15.png](/assets/01KJBZCHRA520E774GE1NMCPY3/image2024-8-1%2020%3A1%3A15.png)

  - 对于每一个proposer的每一个few-shot数据集, 都生成 提示词候选 (proposer.propose_instructions_for_program):

  -     - 某一次生成提示词候选  

      - 列出demo:
        - ![image2024-8-1 23:59:12.png](/assets/01KJBZCHRA520E774GE1NMCPY3/image2024-8-1%2023%3A59%3A12.png)
      - 生成
        - 带上了demo
        - ![image2024-8-2 0:2:35.png](/assets/01KJBZCHRA520E774GE1NMCPY3/image2024-8-2%200%3A2%3A35.png)

    - 使用Optuna进行调优, 一共进行num_batches次尝试. (每次尝试的流程代码: create_objective)
      - 每次, 对每一个predictor, 选取某一个提示词和某一组demo (进行探索), 
        - ![image2024-8-2 0:18:21.png](/assets/01KJBZCHRA520E774GE1NMCPY3/image2024-8-2%200%3A18%3A21.png)
      - 对效果进行评估: 
        - ![image2024-8-2 11:19:13.png](/assets/01KJBZCHRA520E774GE1NMCPY3/image2024-8-2%2011%3A19%3A13.png)

# 注意

Optuna的优势 是在高维空间/大面积空间中进行搜索. 

如果predicator只有一个 (空间维度为1), 且 num_candidates 较小 (空间面积) 的情况下，Optuna 没有优势

(num_batches (尝试次数) 大于 num_candidates时, 不如使用遍历扫描)

MIPROv2 只能对提示词进行一次调优, 而不能继续调优结果继续演进
