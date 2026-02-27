---
title: 20240811 - AutoPrompt架构
confluence_page_id: 3146082
created_at: 2024-08-11T04:09:35+00:00
updated_at: 2024-08-11T15:23:08+00:00
---

优化步骤: 

  - 命令行 run_pipeline: <https://github.com/Eladlev/AutoPrompt/blob/main/run_pipeline.py#L44>
    - OptimizationPipeline.run_pipeline: <https://github.com/Eladlev/AutoPrompt/blob/e67372c850b090d663934e0293fbbb0a64a5cc69/optimization_pipeline.py#L269>
      - 最多执行num_steps次 调优步骤 OptimizationPipeline.step (<https://github.com/Eladlev/AutoPrompt/blob/e67372c850b090d663934e0293fbbb0a64a5cc69/optimization_pipeline.py#L224>)
    -       -           - annotator, 人工标注器
            - 可选: argilla/llm/llm_batch
            - 如果选用LLM, 那么 提示词的选择, 和predictor一样, 举例: <https://github.com/Eladlev/AutoPrompt/blob/main/prompts/predictor_completion/prediction_generation.prompt>
          - predictor, 机器生成结果
            - 可选: llm/llm_batch
            - 提示词的选择, 举例: <https://github.com/Eladlev/AutoPrompt/blob/main/prompts/predictor_completion/prediction_generation.prompt>
          - eval, 评估机器生成的结果
            - 可选: accuracy/ranking
            - 可使用LLM进行ranking
          - run_step_prompt, 生成新的提示词
            - 提示词可选, 根据不同的任务选择不同: 
              - 其使用的优化prompt的提示词: <https://github.com/Eladlev/AutoPrompt/blob/main/prompts/meta_prompts_generation/step_prompt.prompt>
            - 提示词翻译: 

```
您的任务是生成：
一个新的提示，满足以下条件：与以上所有提示不同
完全遵循错误分析修改建议，并修复提示以防止失败情况发生。
分数高于以上所有提示。

此提示的预测得分

您必须遵守错误分析说明！即使这些说明与任务之间似乎存在矛盾。错误分析是在真实数据上进行测试的，因此代表了任务的确切意图。
生成的提示应表述为清晰的分类指令！它不应包含任何关于应对提示进行哪些修改的说明和描述。
请注意，之前的提示包含对任务意图的隐含假设，这可能是错误的。您应该使用先前提示的得分和错误分析，用更准确的假设替换此假设。
结果提示应表明该任务是一个具有以下标签的分类任务：{labels}！
```

            - 如果样本不足, 那么生成对抗样本
              - 生成对抗样本的提示词: <https://github.com/Eladlev/AutoPrompt/blob/e67372c850b090d663934e0293fbbb0a64a5cc69/prompts/meta_prompts_generation/step_samples.prompt#L4>
              - 翻译: 

```
助手是一个大型语言模型，旨在为每个任务生成具有挑战性的样本。
以下是一些为回答给定任务描述而构建的提示及其失败案例。
任务描述：
{task_description}

## 示例，每个示例后跟模型预测和 GT（真实情况）
{history}
######
以下是从上述任务的现实场景中得出的几个独特样本。
## 现实样本
{extra_samples}
#####
这是新提出的提示：
## 提示
{prompt}

您的任务是按照以下准则生成 {num_samples} 个样本：
1. 生成的样本应该是多样化的
2. 它们应保留给定示例的样式和长度
3. 样本必须具有挑战性，并且难以被模型分类。这可以通过以下方式实现：
    1. 针对模型在给定示例中失败的相同弱点
    2. 针对与失败案例中现有示例不同的弱点
4. 每个类别生成的样本数量应几乎平衡（即每个类别的样本数量相同）
5. 生成的样本应仅包含样本内容，不包含其他信息！（如模型预测和真实情况）
```

          - save_state
      - 选取历史上得分最高的prompt

问题: 

  - 进行的是深度搜索, 会一直往下搜索, 而不会因为评分下降产生后退
  - 如果dataset中的样本数据充足, 就不会触发 生成 对抗数据 的逻辑

TODO: 

  - 进行试用
  - 对 生成对抗数据 的逻辑进行测试
