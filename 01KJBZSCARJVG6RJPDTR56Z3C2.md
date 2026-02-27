---
title: 20250407 - 研究SQL优化模型微调 - 让模型对SQL的修改 遵循 改写指令
confluence_page_id: 3801392
created_at: 2025-04-07T02:13:30+00:00
updated_at: 2025-04-15T07:18:32+00:00
---

# 前继

[20250328 - 研究SQL优化模型微调 - 处理错误case[2]]

之前发现: 

  - 模型无法遵循改写指令来改写SQL, 会出现指令多于SQL, 或者SQL多于指令

# 目标

让这个case正确处理

[input.1013.txt](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/input.1013.txt)

# 简化场景

构建测试数据, 测试Qwen2原模型的性能

使用新的脚本, 生成评测数据: /opt/huangyan/sql-ai/make_dataset_rewrite.ipynb

使用eval命令进行评估: 

```
CUDA_VISIBLE_DEVICES=0 swift eval     --model Qwen/Qwen2.5-7B-Instruct-1M     --eval_backend Native     --infer_backend pt     --eval_dataset general_qa     --dataset_args '{"general_qa": {"local_path": "/opt/huangyan/sql-ai/dataset", "subset_list": ["rewrite_sql"]}}'
``` 

评估的结果, 从 /opt/huangyan/sql-ai/eval_output/native/20250407_142458/reviews/Qwen2.5-7B-Instruct-1M/general_qa_rewrite_sql.jsonl 可获取, 类似于: 

````
{"id": "chatcmpl-3c5f08e23ee945abac14a40c9a5d9909", "choices": [{"finish_reason": "stop", "index": 0, "logprobs": null, "message": {"content": "```sql\nSELECT COUNT(*) \nFROM Airlines \nWHERE CANCELLED = 1\n```", "role": "assistant", "tool_calls": null}, "review": {"gold": "```sql\nSELECT COUNT(FlightNumber) FROM Airlines WHERE CANCELLED = 1\n```", "pred": "```sql\nSELECT COUNT(*) \nFROM Airlines \nWHERE CANCELLED = 1\n```", "result": {"rouge-1-r": 0.9, "rouge-1-p": 0.9, "rouge-1-f": 0.899999995, "rouge-2-r": 0.7777777777777778, "rouge-2-p": 0.7777777777777778, "rouge-2-f": 0.7777777727777778, "rouge-l-r": 0.9, "rouge-l-p": 0.9, "rouge-l-f": 0.899999995, "bleu-1": 0.9375, "bleu-2": 0.8666666666666667, "bleu-3": 0.7857142857142857, "bleu-4": 0.6923076923076923}}}], "created": 1744007099, "model": "Qwen2.5-7B-Instruct-1M", "object": "chat.completion", "usage": {"completion_tokens": 20, "prompt_tokens": 537, "total_tokens": 557}, "model_spec": {"api_url": "http://127.0.0.1:8000/v1/chat/completions", "model_id": "Qwen2.5-7B-Instruct-1M", "api_key": "EMPTY"}, "answer_id": "answer-9ea5857ffca387cb67df4911ea1b47f6", "subset_name": "rewrite_sql", "raw_input": {"query": "我需要你根据分析流程中的\"改写指令\"对SQL进行改写, 只输出改写后的SQL, 不要输出其他内容, 不要改变SQL的格式. \n\nSQL:\n----------\nSELECT COUNT(*) FROM Airlines WHERE CANCELLED = 1\n----------\n\n分析流程: \n----------\n让我们像剥洋葱一样，一层层揭开这段SQL的神秘面纱，探究其简洁而直接的逻辑。\n\n1. **最外层查询 (SELECT COUNT(*) FROM ... WHERE ...): 统计取消航班的数量**\n\n这是我们看到的整个查询的最外层结构。它直接作用于`Airlines`表，统计符合特定条件的航班数量。\n\n```sql\nSELECT COUNT(*) \nFROM Airlines \nWHERE CANCELLED = 1\n```\n\n1.1. **FROM 子句 (FROM Airlines): 指定数据来源**\n\n`FROM Airlines`  指定了查询的数据来源是 `Airlines` 表。 这就像考古学家确定了挖掘地点一样，我们接下来的分析都将围绕这张表展开。\n\n```sql\nFROM Airlines\n```\n\n1.2. **WHERE 子句 (WHERE CANCELLED = 1): 筛选取消的航班**\n\n`WHERE CANCELLED = 1`  是查询的核心过滤条件。它筛选出`CANCELLED`列值为1的航班记录。我们可以推测，`CANCELLED`列用于记录航班是否取消，1代表取消，其他值则代表未取消。 就像考古学家根据特定特征筛选文物一样，我们只关注取消的航班。\n\n```sql\nWHERE CANCELLED = 1\n```\n\n1.3. **SELECT 子句 (SELECT COUNT(*)): 统计符合条件的航班数量**\n\n`SELECT COUNT(*)`  用于统计符合`WHERE`子句条件的航班记录数量。`COUNT(*)`函数会计算所有符合条件的行数，并返回一个单一的数值。 就像考古学家统计挖掘出的特定类型文物的数量一样，我们最终得到的是取消航班的总数。\n\n```sql\nSELECT COUNT(*) \n```\n\n\n总而言之，这段SQL的逻辑非常清晰：从`Airlines`表中筛选出`CANCELLED`列值为1的记录，然后统计这些记录的数量，最终返回取消航班的总数。  它就像一个简单的考古挖掘，目标明确，执行高效。\n\n----------\n\n只输出改写后的SQL, 不要输出其他内容, 不要改变SQL的格式:\n", "response": "```sql\nSELECT COUNT(FlightNumber) FROM Airlines WHERE CANCELLED = 1\n```"}, "index": 2, "reviewed": true, "review_id": "review-8656ff8021805d78b94313a7a86f22e9", "reviewer_spec": {"metric": ["AverageBLEU", "AverageRouge"], "reviewer": ["Evaluator"], "revision": ["default"]}, "review_time": 1744007100.3165035}
```` 

评估数据是从BIRD-SQL-data-train数据集的SQL中, 从短到长排序, 每隔50个采样一个. 然后添加一个"改写指令", 进行评估.

用脚本/opt/huangyan/sql-ai/get_reviews.ipynb来获取评估数据中bleu-1 < 0.85的错误记录

  - case总数: 137
  - 评估数据中bleu-1 < 0.85的错误记录: 仅三条错误: [error.0001.txt](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/error.0001.txt)

下一步: 添加两个改写指令, 再进行评估:

  - case总数: 117
  - 评估数据中bleu-1 < 0.85的错误记录: 12条错误: [error.1019.txt](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/error.1019.txt)

添加三个改写指令, 再进行评估:

  - case总数: 163
  - 评估数据中bleu-1 < 0.85的错误记录: 33条错误: [error.1140.txt](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/error.1140.txt)

添加四个改写指令, 再进行评估:

  - case总数: 179
  - 评估数据中bleu-1 < 0.85的错误记录: 62条错误: [error.1310.txt](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/error.1310.txt)

添加五个改写指令, 再进行评估:

  - case总数: 178
  - 评估数据中bleu-1 < 0.85的错误记录: 75条错误: [error.1418.txt](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/error.1418.txt)
  - bleu-1 = 0.8582 (基线)

在 Qwen/Qwen2.5-7B-Instruct-1M 上直接进行DPO, 数据集为 添加1-4个改写指令 中错误的数据构成的DPO, 评估添加5个改写指令的数据, 评估效果: 

| 训练参数 | train loss | 评估bleu-1 | 评估错误个数 |  |
| --- | --- | --- | --- | --- |
| 基线 |  | 0.8582 | 75 |  |
| LR=1e-4, beta=0.1, temperature=0.6, epoch=4,gradient_accumulation_steps=8 | 0.29014587 | 0.8301 |  |  |
| LR=1e-4, beta=0.1, temperature=0.9, epoch=4, gradient_accumulation_steps=8 | 0.28813171 | 0.8346 |  |  |
| LR=1e-4, beta=0.1, temperature=0.9, epoch=8, gradient_accumulation_steps=8 | 0.15227661 | 0.7048 | 69 | ![image2025-4-8 21:23:34.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-8%2021%3A23%3A34.png)![image2025-4-8 21:23:42.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-8%2021%3A23%3A42.png) |
| LR=1e-3, beta=0.1, temperature=0.9, epoch=8, gradient_accumulation_steps=8 | 0.01690121 | 0.8791 | 43 | ![image2025-4-8 21:51:52.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-8%2021%3A51%3A52.png)![image2025-4-8 21:52:4.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-8%2021%3A52%3A4.png)SOTALR增大, reward margin变大, 主要是rejected的评分更低.评估效果较好 |
| LR=1e-3, beta=0.1, temperature=0.9, epoch=16, gradient_accumulation_steps=8 | 0.00190039 | 0.8236 | 52 | ![image2025-4-9 13:23:1.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2013%3A23%3A1.png)![image2025-4-9 13:23:12.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2013%3A23%3A12.png)与SOTA相比, 增加epoch, 导致效果变差 |
| LR=1e-3, beta=0.5, temperature=0.9, epoch=8, gradient_accumulation_steps=8 | 0.02190464 | 0.8801 | 45 | ![image2025-4-9 14:35:49.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2014%3A35%3A49.png)![image2025-4-9 14:35:57.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2014%3A35%3A57.png)与SOTA相比, 增大了beta (KL散度约束增强), 发现rewards差异变大(正负方向都变大),也就是说beta=0.5更关注于细节, 这个粒度的细节获得了更大的奖励 |
| LR=1e-4, beta=0.5, temperature=0.9, epoch=16, gradient_accumulation_steps=8 | 0.00392663 | 0.7568 | 60 | ![image2025-4-9 16:0:30.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2016%3A0%3A30.png)![image2025-4-9 16:1:5.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2016%3A1%3A5.png)与SOTA相比, 增大了beta (KL散度约束增强), 增大了epoch增大了epoch使得效果变差. 同时reward rejected变得更低, chosen没有变化 |
| LR=1e-4, beta=0.1, temperature=0.9, epoch=16, gradient_accumulation_steps=8 | 0.02704086 | 0.7996 | 59 | ![image2025-4-8 21:23:3.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-8%2021%3A23%3A3.png)![image2025-4-8 21:23:11.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-8%2021%3A23%3A11.png)与SOTA相比, LR继续增大, 效果变差 |
| LR=1e-4, beta=0.1, temperature=0.9, epoch=16, gradient_accumulation_steps=16 | 0.13616638 | 0.7375 | 63 | ![image2025-4-8 21:22:6.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-8%2021%3A22%3A6.png)![image2025-4-8 21:22:19.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-8%2021%3A22%3A19.png)与SOTA相比, LR继续增大, gradient_accumulation_steps增大, 效果变差 |
| LR=1e-3, beta=0.7, temperature=0.9, epoch=8, gradient_accumulation_steps=8 | 0.04940729 | 0.9127 | 33 | ![image2025-4-9 16:21:48.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2016%3A21%3A48.png)![image2025-4-9 16:21:59.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2016%3A21%3A59.png) 与SOTA + beta=0.5 相比, beta继续增大, 效果变好其中: reward_chosen在变大, reward_rejected的变化幅度不大 |
| LR=1e-3, beta=0.9, temperature=0.9, epoch=8, gradient_accumulation_steps=8| 0.04741592| 0.9164| 29| ![image2025-4-9 16:36:29.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2016%3A36%3A29.png)![image2025-4-9 16:36:38.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2016%3A36%3A38.png)与SOTA + beta=0.7 相比, beta继续增大, 效果轻微变好但: reward两个方向变化幅度都不小, margin扩大
| LR=1e-3, beta=1.2, temperature=0.9, epoch=8, gradient_accumulation_steps=8| 0.04771233| 0.9028| 31| ![image2025-4-9 19:13:56.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2019%3A13%3A56.png)![image2025-4-9 19:14:4.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2019%3A14%3A4.png)与SOTA + beta=0.9 相比, beta继续增大, 效果没有增强但: reward两个方向变化幅度都不小, margin继续扩大 增加验证集的reward趋势: ![image2025-4-9 23:46:41.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2023%3A46%3A41.png)![image2025-4-9 23:46:49.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2023%3A46%3A49.png)![image2025-4-9 23:48:31.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-9%2023%3A48%3A31.png)
  
# 举例说明DPO的分析

对(LR=1e-3, beta=1.2, temperature=1.2, epoch=8,gradient_accumulation_steps=8) 的进一步分析:

eval/rewards/chosen 的趋势 和 eval/loss一致, 但 eval/rewards/rejected一直在下降 (导致margin一直在上升), 也就是说DPO的过程为了追求margin, 而过度惩罚了负向案例 (惩罚错了方向), 这个惩罚方向轻微拉低了chosen的效果

进一步分析: beta=1.2值比较高, 其对KL的惩罚比较重(要求模型遵循原始输出, 创新力降低), 因为创新力降低, 所以突破chosen的天花板比较困难, 被迫压制rejected

![image2025-4-10 0:2:41.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%200%3A2%3A41.png)![image2025-4-10 0:2:47.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%200%3A2%3A47.png)![image2025-4-10 0:2:51.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%200%3A2%3A51.png)

![image2025-4-10 0:2:31.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%200%3A2%3A31.png)

![image2025-4-10 0:2:13.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%200%3A2%3A13.png)

![image2025-4-10 0:1:40.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%200%3A1%3A40.png)![image2025-4-10 0:1:49.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%200%3A1%3A49.png)![image2025-4-10 0:1:58.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%200%3A1%3A58.png)

阶段结论: 

  - DPO训练中, beta参数决定了奖励针对的粒度 (针对粗粒度奖励, 还是针对细粒度信息奖励)
  - 需要评估集的reward, 来评估奖励的方向 (正向上升多还是负向减少多), 来避免过度压制负例, 而不真正提升loss

# 生成更多训练数据, 尝试突破瓶颈

在原始数据上, 隔25个数据采样一次 (原来是隔50个数据采样一次), 选取Qwen模型的错误case作为DPO

DPO数据量: 

  - 1个变更: 16 (隔10个数据采样一次, 生成了超量数据, 属于脚本错误)
  - 2个变更: 68 (隔10个数据采样一次, 生成了超量数据, 属于脚本错误)
  - 3个变更: 64
  - 4个变更: 103
  - 5个变更: 132
  - 6个变更: 145

| 训练参数 | 评估bleu-1 | 评估错误个数 | train reward图像 | eval reward图像 |
| --- | --- | --- | --- | --- |
| LR=1e-3, beta=0.9,temperature=0.9, epoch=8,gradient_accumulation_steps=8 (数据: 训练1-4, 验证5) | 0.8862 | 72 | ![image2025-4-10 19:3:26.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A3%3A26.png)![image2025-4-10 19:3:37.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A3%3A37.png)![image2025-4-10 19:3:45.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A3%3A45.png) | ![image2025-4-10 19:3:16.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A3%3A16.png)![image2025-4-10 19:1:14.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A1%3A14.png)![image2025-4-10 19:1:25.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A1%3A25.png) chosen在降低, 但变化幅度不大, 无法突破优化的瓶颈.可以看到loss和chosen的关系考虑降低beta, 增加创新能力 |
| LR=1e-3, beta=0.5,temperature=0.9, epoch=8,gradient_accumulation_steps=8 (数据: 训练1-4, 验证5) | 0.8042 | 100 | ![image2025-4-10 19:33:43.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A33%3A43.png)![image2025-4-10 19:33:33.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A33%3A33.png)![image2025-4-10 19:34:0.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A34%3A0.png) | ![image2025-4-10 19:34:32.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A34%3A32.png)![image2025-4-10 19:35:4.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A35%3A4.png)![image2025-4-10 19:35:26.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A35%3A26.png) 降低beta=0.5 效果并不好, 可以看到loss和chosen的关系 |
| LR=1e-3, beta=1.5,temperature=0.9, epoch=8,gradient_accumulation_steps=8 (数据: 训练1-4, 验证5) | 0.6475 | 147 | ![image2025-4-10 19:55:19.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A55%3A19.png) |  |
| ![image2025-4-10 19:55:42.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A55%3A42.png)![image2025-4-10 19:55:49.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A55%3A49.png) | ![image2025-4-10 19:55:56.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A55%3A56.png)![image2025-4-10 19:56:5.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A56%3A5.png)![image2025-4-10 19:56:16.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2019%3A56%3A16.png) 升高beta=1.5 效果更差 |  |  |  |
| LR=1e-4, beta=0.9,temperature=0.9, epoch=8,gradient_accumulation_steps=8 (数据: 训练1-4, 验证5) | 0.871 | 70 | ![image2025-4-10 20:22:2.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2020%3A22%3A2.png)![image2025-4-10 20:22:9.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2020%3A22%3A9.png)![image2025-4-10 20:22:16.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2020%3A22%3A16.png) | ![image2025-4-10 20:22:29.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2020%3A22%3A29.png)![image2025-4-10 20:22:36.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2020%3A22%3A36.png)![image2025-4-10 20:22:44.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2020%3A22%3A44.png)chosen开始持平, 并没有出现下降.整体效果也跟LR=1e-3持平, 尝试延长epoch |
| LR=1e-4, beta=0.9,temperature=0.9, epoch=12,gradient_accumulation_steps=8 (数据: 训练1-4, 验证5) | 0.7799 | 102 | ![image2025-4-10 22:14:32.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A14%3A32.png)![image2025-4-10 22:14:38.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A14%3A38.png)![image2025-4-10 22:14:43.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A14%3A43.png) | ![image2025-4-10 22:14:53.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A14%3A53.png)![image2025-4-10 22:15:3.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A15%3A3.png)![image2025-4-10 22:15:14.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A15%3A14.png) 扩大epoch, 其无法突破reward chosen的极限, 后面强行压制rejected, 导致整体效果变糟糕 |
| LR=3e-4, beta=0.9,temperature=0.9, epoch=8,gradient_accumulation_steps=8 (数据: 训练1-4, 验证5) | 0.7969 | 95 | ![image2025-4-10 22:17:48.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A17%3A48.png)![image2025-4-10 22:17:53.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A17%3A53.png)![image2025-4-10 22:17:58.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A17%3A58.png) | ![image2025-4-10 22:18:10.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A18%3A10.png)![image2025-4-10 22:18:16.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A18%3A16.png)![image2025-4-10 22:18:27.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A18%3A27.png) 随着LR变大, 其更快地到达了极值点, 但无法突破 |
| LR=1e-4, beta=0.5,temperature=0.9, epoch=8,gradient_accumulation_steps=8 (数据: 训练1-4, 验证5) | 0.8551 | 73 | ![image2025-4-10 22:20:20.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A20%3A20.png)![image2025-4-10 22:20:27.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A20%3A27.png)![image2025-4-10 22:20:35.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A20%3A35.png) | ![image2025-4-10 22:20:51.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A20%3A51.png)![image2025-4-10 22:20:59.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A20%3A59.png)![image2025-4-10 22:21:6.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A21%3A6.png) |
| LR=1e-4, beta=0.5,temperature=0.9, epoch=12,gradient_accumulation_steps=8 (数据: 训练1-4, 验证5) | 0.8244 | 89 | ![image2025-4-10 23:7:1.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2023%3A7%3A1.png)![image2025-4-10 23:7:8.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2023%3A7%3A8.png)![image2025-4-10 23:7:17.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2023%3A7%3A17.png) | ![image2025-4-10 23:9:10.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2023%3A9%3A10.png)![image2025-4-10 23:9:18.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2023%3A9%3A18.png)![image2025-4-10 23:9:24.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2023%3A9%3A24.png) |
| LR=1e-4, beta=0.2,temperature=0.9, epoch=8,gradient_accumulation_steps=8 (数据: 训练1-4, 验证5) | 0.8135 | 90 | ![image2025-4-10 22:55:57.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A55%3A57.png)![image2025-4-10 22:56:2.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A56%3A2.png)![image2025-4-10 22:56:7.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A56%3A7.png) | ![image2025-4-10 22:57:27.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A57%3A27.png)![image2025-4-10 22:57:35.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A57%3A35.png)![image2025-4-10 22:57:41.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-10%2022%3A57%3A41.png) reward和loss没到过拟合, 尝试调大LR? |
  
发现评估时, 出现了一些GT的输出, 仅是"```", 需要排查数据问题, 修正后, 循环穷举参数组合: 

  - betas=(1.3 1.1 0.9 0.7 0.5)
  - learning_rates=(1e-4 1e-5)
  - epochs=(8 12)
  - temperatures=(0.9)

结果: 

==== [Experiment] beta=1.3 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9213
  - 错误数: 49

==== [Experiment] beta=1.3 | lr=1e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.91

==== [Experiment] beta=1.3 | lr=1e-5 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.8832

==== [Experiment] beta=1.3 | lr=1e-5 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.8922

==== [Experiment] beta=1.1 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9202
  - 错误数: 48  

==== [Experiment] beta=1.1 | lr=1e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.8939

==== [Experiment] beta=1.1 | lr=1e-5 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.8815

==== [Experiment] beta=1.1 | lr=1e-5 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.8919

==== [Experiment] beta=0.9 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9209
  - 错误数: 48  

==== [Experiment] beta=0.9 | lr=1e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.8981

==== [Experiment] beta=0.9 | lr=1e-5 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.883

==== [Experiment] beta=0.9 | lr=1e-5 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.894

==== [Experiment] beta=0.7 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9223
  - 错误数: 51  

==== [Experiment] beta=0.7 | lr=1e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.9039

==== [Experiment] beta=0.7 | lr=1e-5 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.8852

==== [Experiment] beta=0.7 | lr=1e-5 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.892

==== [Experiment] beta=0.5 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9222
  - 错误数: 49  

==== [Experiment] beta=0.5 | lr=1e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.8966

==== [Experiment] beta=0.5 | lr=1e-5 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.8867

==== [Experiment] beta=0.5 | lr=1e-5 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.8929

粗筛, 结果比较好的都是 "lr=1e-4 | epochs=8", beta的影响不大

  - 猜测: beta影响不大的原因: 原来有GT SQL为空的情况, 对模型挑战比较大(什么时候应该输出空), 需要比较跳跃的思维. 去除GT SQL为空的情况后, 挑战就变小, 不需要太高的创造能力

### 固定beta=0.5, 查看LR和epoch的影响:

==== [Experiment] beta=0.5 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9222
  - 错误数: 49  

  - ![image2025-4-11 10:57:12.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2010%3A57%3A12.png)![image2025-4-11 10:57:18.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2010%3A57%3A18.png)![image2025-4-11 10:57:24.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2010%3A57%3A24.png)

![image2025-4-11 10:57:32.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2010%3A57%3A32.png)![image2025-4-11 10:57:41.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2010%3A57%3A41.png)![image2025-4-11 11:0:53.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A0%3A53.png)

  

==== [Experiment] beta=0.5 | lr=1e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.8966
  - 错误数: 65
  - ![image2025-4-11 11:1:55.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A1%3A55.png)![image2025-4-11 11:2:0.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A2%3A0.png)![image2025-4-11 11:2:12.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A2%3A12.png)

![image2025-4-11 11:2:23.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A2%3A23.png)![image2025-4-11 11:2:36.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A2%3A36.png)![image2025-4-11 11:2:44.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A2%3A44.png)

==== [Experiment] beta=0.5 | lr=1e-5 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.8867
  - 错误数: 97
  - ![image2025-4-11 11:5:2.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A5%3A2.png)![image2025-4-11 11:5:15.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A5%3A15.png)![image2025-4-11 11:5:21.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A5%3A21.png)

![image2025-4-11 11:5:32.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A5%3A32.png)![image2025-4-11 11:5:39.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A5%3A39.png)![image2025-4-11 11:5:42.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2011%3A5%3A42.png)

==== [Experiment] beta=0.5 | lr=1e-5 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.8929
  - 错误数: 82
  - ![image2025-4-11 13:1:49.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A1%3A49.png)![image2025-4-11 13:1:56.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A1%3A56.png)![image2025-4-11 13:2:1.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A2%3A1.png)

![image2025-4-11 13:2:14.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A2%3A14.png)![image2025-4-11 13:2:21.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A2%3A21.png)![image2025-4-11 13:2:26.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A2%3A26.png)

  - 观察: 
    - LR=1e-5, 看上去曲线 eval/rewards/chosen 还会继续上升
    - LR=1e-4, epochs=12中, eval/rewards/chosen 已经产生了极值和回落
      - 测试这个场景(==== [Experiment] beta=0.5 | lr=1e-4 | epochs=12 | temperature=0.9 ====)的best epoch的表现: 
        - bleu-1=0.9193, 错误数: 53
        - 跟epoch=8时的效果差不多. 也就是说: 可以通过best epoch来进行评估 (loss和bleu-1和真实的错误数 之间有正相关性)
    - 从以上得到结论是: 扩大LR可以快速找到最优解, epoch+best 评估, 可以不进行多种epoch实验

还有一些空的SQL在训练数据中, 进行了删除

### 固定 lr=1e-4 | epochs=8 | temperature=0.9 , 研究beta的变化

==== [Experiment] beta=1.3 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9213
  - 错误数: 49
  - ![image2025-4-11 13:43:8.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A43%3A8.png)![image2025-4-11 13:43:13.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A43%3A13.png)![image2025-4-11 13:43:18.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A43%3A18.png)

![image2025-4-11 13:43:27.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A43%3A27.png)![image2025-4-11 13:43:33.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A43%3A33.png)![image2025-4-11 13:43:38.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A43%3A38.png)

==== [Experiment] beta=1.1 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9202
  - 错误数: 48
  - ![image2025-4-11 13:44:50.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A44%3A50.png)![image2025-4-11 13:44:55.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A44%3A55.png)![image2025-4-11 13:45:0.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A45%3A0.png)

![image2025-4-11 13:45:7.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A45%3A7.png)![image2025-4-11 13:45:13.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A45%3A13.png)![image2025-4-11 13:45:18.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A45%3A18.png)

==== [Experiment] beta=0.9 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9209
  - 错误数: 48
  - ![image2025-4-11 13:49:48.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A49%3A48.png)![image2025-4-11 13:49:54.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A49%3A54.png)![image2025-4-11 13:50:1.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A50%3A1.png)

![image2025-4-11 13:50:9.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A50%3A9.png)![image2025-4-11 13:50:15.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A50%3A15.png)![image2025-4-11 13:50:20.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A50%3A20.png)

==== [Experiment] beta=0.7 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9223
  - 错误数: 51
  - ![image2025-4-11 13:50:59.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A50%3A59.png)![image2025-4-11 13:51:4.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A51%3A4.png)![image2025-4-11 13:51:14.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A51%3A14.png)

![image2025-4-11 13:51:21.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A51%3A21.png)![image2025-4-11 13:51:26.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A51%3A26.png)![image2025-4-11 13:51:32.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A51%3A32.png)

==== [Experiment] beta=0.5 | lr=1e-4 | epochs=8 | temperature=0.9 ====

  - bleu-1 score: 0.9222
  - 错误数: 49
  - ![image2025-4-11 13:52:14.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A52%3A14.png)![image2025-4-11 13:52:19.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A52%3A19.png)![image2025-4-11 13:52:24.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A52%3A24.png)

![image2025-4-11 13:52:31.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A52%3A31.png)![image2025-4-11 13:52:39.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A52%3A39.png)![image2025-4-11 13:52:44.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2013%3A52%3A44.png)

### 观察: 

  - 随着beta变化, 几根曲线的形状都一样, rewards的阈值跟beta有相关性, 但趋势是不变的
    - 与之前的结论相比: 之前有空SQL数据, 导致需要创新能力来推理, 所以不同的beta曲线会不同(随创新能力不同, 训练中找到了不同的突破方向)
    - 现在去除了空SQL数据, 不需要太多的创新能力来推理, 所以不同的beta收敛的方向差不多, 曲线相似

### 下一步

  - 继续清理了空SQL数据
  - 忽略beta的变化, 固定beta=0.5
  - 固定epoch=12, 取best model进行评估
  - 变量还有LR和temperature
  - 研究错误数据的特征

==== [Experiment] beta=0.5 | lr=3e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.8741
  - 错误数: 68
  - ![image2025-4-11 15:19:28.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A19%3A28.png)![image2025-4-11 15:19:35.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A19%3A35.png)![image2025-4-11 15:19:41.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A19%3A41.png)

![image2025-4-11 15:19:49.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A19%3A49.png)![image2025-4-11 15:19:55.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A19%3A55.png)![image2025-4-11 15:19:59.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A19%3A59.png)

  - 很快产生了过拟合

==== [Experiment] beta=0.5 | lr=1e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.9283
  - 错误数: 52
  - ![image2025-4-11 15:21:0.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A21%3A0.png)![image2025-4-11 15:21:6.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A21%3A6.png)![image2025-4-11 15:21:10.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A21%3A10.png)

![image2025-4-11 15:21:22.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A21%3A22.png)![image2025-4-11 15:21:28.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A21%3A28.png)![image2025-4-11 15:21:32.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A21%3A32.png)

==== [Experiment] beta=0.2 | lr=3e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.9294
  - 错误数: 51
  - ![image2025-4-11 15:22:3.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A22%3A3.png)![image2025-4-11 15:22:7.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A22%3A7.png)![image2025-4-11 15:22:12.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A22%3A12.png)

![image2025-4-11 15:22:19.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A22%3A19.png)![image2025-4-11 15:22:27.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A22%3A27.png)![image2025-4-11 15:22:32.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A22%3A32.png)

  - LR=3e-4, 迅速发生过拟合. beta=0.2, 过拟合时更早更激烈?

==== [Experiment] beta=0.2 | lr=1e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.9277
  - 错误数: 52
  - ![image2025-4-11 15:24:12.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A24%3A12.png)![image2025-4-11 15:24:19.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A24%3A19.png)![image2025-4-11 15:24:23.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A24%3A23.png)

![image2025-4-11 15:24:35.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A24%3A35.png)![image2025-4-11 15:24:40.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A24%3A40.png)![image2025-4-11 15:24:45.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-11%2015%3A24%3A45.png)

观察: 

  - beta继续缩小, 不影响极值, 会更快的达到过拟合. 可以忽略这一部分的作用
  - LR=3e-4, 会更快(过早)达到过拟合, 显然LR过大.

错误分析: 发现有一些GT错误, 引入了不在指令中的变更, 比如: (如下示例中total_credit没有在指令中)

````
{"query": "我需要你根据分析流程中的\"改写指令\"对SQL进行改写, 只输出改写后的SQL, 不要输出其他内容, 不要改变SQL的格式. 

SQL:
----------
SELECT country, COUNT(customerNumber) FROM customers GROUP BY country
----------

分析流程: 
----------
让我们像剥洋葱一样，一层层揭开这段SQL的神秘面纱，探究其结构和逻辑。

1. **最外层查询 (SELECT ... FROM ... GROUP BY ...): 统计每个国家的顾客数量**

这是我们看到的整个查询的最外层结构。它从`customers`表中提取数据，按国家分组统计顾客数量，并最终输出每个国家及其对应的顾客数量。其中有计算列：
    - 计算列 `COUNT(customerNumber)`: 这一列使用了`COUNT()`函数，用于统计每个国家的顾客数量。`customerNumber`作为`COUNT()`函数的参数，表示统计非空`customerNumber`值的数量。`GROUP BY country`子句确保了`COUNT()`函数对每个不同的国家分别进行计数。
    
    ```sql
SELECT country, COUNT(customerNumber) 
FROM customers 
GROUP BY country;
```

   改写指令: 将COUNT(customerNumber)重命名为customer_count。
   
   改写指令: 添加ORDER BY customer_count DESC排序。
   
   1.1. **数据源 (FROM customers):  顾客信息表**

最外层查询的数据源自`customers`表。我们可以推测，这张表存储了所有顾客的信息，其中至少包含`country`（国家）和`customerNumber`（顾客编号）两列。`country`列用于区分不同的国家，`customerNumber`列用于标识不同的顾客。

```sql
FROM customers
```

   改写指令: 添加WHERE子句，筛选出creditLimit大于10000的顾客。
   
   改写指令: 选择country和customerName两列。
   
   1.2. **分组 (GROUP BY country): 按国家分组**

`GROUP BY country`子句将`customers`表中的数据按照`country`列的值进行分组。这意味着，具有相同`country`值的记录会被归为一组。这为后续的聚合操作（例如`COUNT()`函数）提供了分组依据。

```sql
GROUP BY country
```

总而言之，这段SQL的逻辑非常清晰：从`customers`表中提取数据，按国家分组，然后统计每个国家的顾客数量。最终输出的结果是一个包含两列的表格：`country`列表示国家，`COUNT(customerNumber)`列表示该国家的顾客数量。

   改写指令: 在GROUP BY country 后面添加 HAVING customer_count > 5。
   
   ----------

只输出改写后的SQL, 不要输出其他内容, 不要改变SQL的格式(尤其不要进行SQL格式化, 不要添加额外的空行), 输出需要包裹在\"```sql\"块中:
", "response": "```sql
SELECT country, COUNT(customerNumber) AS customer_count, SUM(creditLimit) AS total_credit
FROM customers
WHERE creditLimit > 10000
GROUP BY country
HAVING customer_count > 5
ORDER BY customer_count DESC;
```"} 
```` 

需要在生成数据时, 引入校验机制. 重新修正数据.

修正数据后, 进行测试, 同时测试LR和temperature的影响: 

==== [Experiment] beta=0.5 | lr=1e-4 | epochs=12 | temperature=0.9 ====

  - bleu-1 score: 0.9427
  - 错误数: 38
  - ![image2025-4-12 14:54:57.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A54%3A57.png)![image2025-4-12 14:55:2.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A55%3A2.png)![image2025-4-12 14:55:6.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A55%3A6.png)

![image2025-4-12 14:55:15.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A55%3A15.png)![image2025-4-12 14:55:24.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A55%3A24.png)![image2025-4-12 14:55:27.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A55%3A27.png)

==== [Experiment] beta=0.5 | lr=1e-4 | epochs=12 | temperature=1.5 ====

  - bleu-1 score: 0.9449
  - 错误数: 36
  - ![image2025-4-12 14:56:38.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A56%3A38.png)![image2025-4-12 14:56:45.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A56%3A45.png)![image2025-4-12 14:56:58.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A56%3A58.png)

![image2025-4-12 14:57:46.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A57%3A46.png)![image2025-4-12 14:57:54.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A57%3A54.png)![image2025-4-12 14:58:4.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-12%2014%3A58%3A4.png)

  - 观察: 这组数据中, temperature的变化几乎没什么影响

==== [Experiment] beta=0.5 | lr=5-e5 | epochs=12 | temperature=0.9 ====  
==== [Experiment] beta=0.5 | lr=5-e5 | epochs=12 | temperature=1.5 ====

对(==== [Experiment] beta=0.5 | lr=1e-4 | epochs=12 | temperature=0.9 ====)的错误进行分析: 

  - review文件: [reviews_2025-04-12_00-59-19.log](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/reviews_2025-04-12_00-59-19.log)
  - 增加分析结论: [reviews_2025-04-12_00-59-19.analyzed.log](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/reviews_2025-04-12_00-59-19.analyzed.log)
  - 错误分类: 
    - 其中有一半错误, 跟 [表名前缀/表名别名是否存在, 默认排序方向, 列顺序/条件顺序] 有关. 另一半是缺失错误.

```
分析: 减少表名前缀, 调整了条件顺序
分析: 减少表名前缀, 增加了排序方向
分析: 别名错误
分析: 别名错误
分析: 别名错误
分析: 增加了别名
分析: 增加了别名引号
分析: 增加了表名前缀
分析: 增加表名前缀
分析: 增加表名前缀
分析: 增加表名前缀
分析: 增加表名前缀
分析: 增加表名前缀
分析: 增加表名前缀
分析: 增加表名前缀
分析: 增加表名前缀
分析: 增加表名前缀, 增加排序方向
分析: 缺少表名前缀, 调整了列顺序
分析: 缺少表别名
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
分析: 错误
``` 

需要对训练数据进行修订, 让: 

  - 表名前缀 匹配 原SQL或改写要求
  - 默认排序顺序 匹配 原SQL或改写要求
  - 表别名 匹配 原SQL或改写要求
  - 条件顺序和列顺序, 如无特殊要求, 增加到最后
  - SQL格式应遵循原SQL格式, 如无特殊要求, 不进行格式化

修正数据: commit: 9f1d05213dc650b8815e7c8febe9a9130b04070c. 重新进行训练: 

  - ==== [Experiment] beta=0.5 | lr=5e-5 | epochs=12 | temperature=0.9 ====

    - bleu-1 score: 0.9462
    - review error count: 33
    - ![image2025-4-13 17:50:0.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A50%3A0.png)![image2025-4-13 17:50:5.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A50%3A5.png)![image2025-4-13 17:50:12.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A50%3A12.png)

![image2025-4-13 17:50:24.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A50%3A24.png)![image2025-4-13 17:50:29.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A50%3A29.png)![image2025-4-13 17:50:33.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A50%3A33.png)

  - ==== [Experiment] beta=0.5 | lr=5e-5 | epochs=12 | temperature=1.5 ====

    - bleu-1 score: 0.945
    - review error count: 32
    - ![image2025-4-13 17:50:56.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A50%3A56.png)![image2025-4-13 17:51:22.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A51%3A22.png)![image2025-4-13 17:51:26.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A51%3A26.png)

![image2025-4-13 17:51:37.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A51%3A37.png)![image2025-4-13 17:51:42.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A51%3A42.png)![image2025-4-13 17:51:49.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2017%3A51%3A49.png)

temperature的调整, 对结果没有影响.

去除change_1和change_2的训练数据 (没经过清洗), 观察训练效果: (参数: betas=0.5,learning_rates=5e-5,epochs=12,temperatures=0.6):

  - bleu-1 score: 0.9459
  - review error count: 28
  - 去除change_1和change_2的训练数据, 没有负向影响

![image2025-4-13 22:14:56.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2022%3A14%3A56.png)![image2025-4-13 22:15:3.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2022%3A15%3A3.png)![image2025-4-13 22:15:8.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2022%3A15%3A8.png)

![image2025-4-13 22:15:31.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2022%3A15%3A31.png)![image2025-4-13 22:15:38.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2022%3A15%3A38.png)![image2025-4-13 22:15:42.png](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/image2025-4-13%2022%3A15%3A42.png)

  - 看起来eval/loss并没有过拟合, 得尝试调大LR

在扩大验证集数量 (让eval/loss更准, 使得选择best_model时更精确)

  - bleu-1 score: 0.9435
  - review error count: 31
  - 扩大验证集数量后, best_model不再是last_model, 但loss差异很小 ( 0.09561463 v.s 0.09596202 )

对评估结果进行查看: [reviews_2025-04-13_18-12-05.log](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/reviews_2025-04-13_18-12-05.log), 其中:

  - 存在漏指令的情况
  - 仍存在 单行/多行 不对的情况, 可能是训练数据仍有遗漏, 但可以留到后面再解决

目前的训练数据只有: change_3和change_4 (改变了三处和改变了四处)的数据, 数据量分别是: 69 和 97. 

将change_5也介入训练, 评估change_6试试.

  - ==== [Experiment] beta=0.5 | lr=1e-4 | epochs=12 | temperature=0.6 ====
    - bleu-1 score: 0.943
    - review error count: 39
  - ==== [Experiment] beta=0.5 | lr=1e-4 | epochs=16 | temperature=0.6 ====
    - bleu-1 score: 0.9377
    - review error count: 42
  - ==== [Experiment] beta=0.5 | lr=5e-5 | epochs=12 | temperature=0.6 ====
    - bleu-1 score: 0.9405
    - review error count: 37
  - ==== [Experiment] beta=0.5 | lr=5e-5 | epochs=16 | temperature=0.6 ====
    - bleu-1 score: 0.9358
    - review error count: 46

都出现了过拟合的情况. 进行错误分析: (==== [Experiment] beta=0.5 | lr=1e-4 | epochs=12 | temperature=0.6 ====):

[reviews_2025-04-14_01-03-43.log](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/reviews_2025-04-14_01-03-43.log) (分析了一部分), 错误分布如下: 

# 可接受的差异  
分析: 缺失了表别名前缀 (可接受的差异)  
分析: 缺失了表别名前缀 (可接受的差异)  
分析: 增加了表别名前缀 (可接受的差异), 列顺序错误 (可接受的差异)  
分析: 增加了表名前缀 (可接受的差异)  
分析: 注释错误 (可接受的差异)  
分析: 增加了表名前缀的引号 (可接受的差异)  
分析: 对于指令"改写指令: 在WHERE子句中添加条件，要求`CREATED_DATE` (假设存在该列) 在过去的一周内。", 计算的方法不同 (可接受的差异)  
分析: 对指令"改写指令: 添加一个名为YearRange的列，显示年份范围。"的不同实现 (可接受的差异)  
分析: 对指令"改写指令: 为 `T1.birthCountry` 添加别名 `BirthCountry`"的误解 (可接受的差异)  
分析: 指令"改写指令: 移除university表的别名T1。"的执行, GT错误增加了另一个别名. pred是正确的 (可接受的差异)  
分析: 指令使用了错误的表名"改写指令: 在SELECT语句中添加 customers.c_name AS CustomerName，显示客户姓名。", pred进行了正确的修正 (可接受的差异)  
分析: 指令错误"改写指令: 为T2.device_id添加别名device_id_from_phone_table。" (可接受的差异)  
分析: 指令"改写指令: 在WHERE子句中添加条件，要求投票数量大于100。". 对这个指令理解出现错误 (可接受的差异)  
分析: 输出格式错误, SQL正确 (可接受的差异)  
分析: 指令"改写指令: 添加一个条件：`day_id` 小于等于 5。" 与现有条件冲突 (GT错误) (可接受的差异)

# 遗漏指令  
分析: 遗漏指令"改写指令: 选择 T2 表中的 `playerID` 列，并为其添加别名 `ScoringPlayerID`。";   
分析: 没有遵循指令"将age列的数据类型隐式转换为数值型", 缺少隐式转换  
分析: 遗漏指令"改写指令: 添加一个新的过滤条件：CampaignID = 123"  
分析: 遗漏指令"改写指令: 添加一列 `GameDate`，并将其格式化为 'YYYY-MM-DD' 的字符串。"

# 多删除了列  
分析: 多删除了Comments列  
分析: 多删除了middle和last列

# GROUP BY  
分析: 指令"改写指令: 添加 COUNT(c.ID) AS CallCount 列来统计每个方法的调用次数", 没有增加GROUP BY  
分析: 多了一列store_nbr, 应该是指令"改写指令: 添加GROUP BY store_nbr, 对商店编号进行分组统计。"的副作用  
分析: 对指令"改写指令: 在SELECT语句中添加group_concat(T1.gender) AS genders，用于获取每个设备型号对应的所有性别，并使用逗号分隔。"的处理, 缺少了GROUP BY子句

# MISC  
分析: 错误处理了"改写指令: 将`date`列重命名为`weather_date`", 应当是重命名但处理成了别名  
分析: 指令"改写指令: 添加条件，ProductID IN (912, 913, 914)。", 与现有条件"WHERE ProductID = 912" 之间冲突解决错误  
分析: 对指令"改写指令: 为T2.Client_ID 添加别名 EventClientID。", 并没有增加列

扩大生成DPO时的temperature, 从0调整到0.6, 企图生成更多样的DPO负例数据. 

(另外: 发现训练后的评估脚本, 没有使用best_model, 而一直使用了last_model, 本次进行了修正)

进行训练的结果: 

  - 生成DPO数据的数量
    - change_3: 68
    - change_4: 97
    - change_5: 133
  - 训练 (==== [Experiment] beta=0.5 | lr=1e-4 | epochs=12 | temperature=0.6 ====):
    - best_model:
      - bleu-1 score: 0.9391
      - review error count: 40
    - last_model:
      - bleu-1 score: 0.9425
      - review error count: 39

未见明显区别

扩大eval的temperature, 分别使用0和2.0, 然后进行去重合并, 数据量: 

  - change_3: 71
  - change_4: 105
  - change_5: 141

增量很小

重申出现的错误 (在评估脚本中增加LLM判断, 排除SQL中的别名差异等), 剩下的错误是28个: [1649.txt](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/1649.txt)

分析其中错误, 大部分错误都不是缺失指令, 而是对指令的误解, 有GT错误也有pred错误.

考虑停在此处, 从这个模型上进行优化的微调, 查看指令遵循的效果

model: /opt/huangyan/sql-ai/output/Qwen2.5-7B-Instruct-1M/v95-20250414-150059/checkpoint-44

data: /opt/huangyan/sql-ai/dataset/20250414

commit: 1b2b209ec226915c423469bb78d1b938a1fcf9be

# 回到SQL优化任务, 查看指令遵循的效果

推理效果: [2025-04-14_23-09-21.log](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/2025-04-14_23-09-21.log)

仍然存在SQL修改和指令不匹配的情况: 

  - 删除列的指令没有被遵守
  - SQL变更中有修改列名, 但没有对应指令

需要调整DPO的数据, 引入"删除列的指令没有被遵守"这种情况试试

  - 用/opt/huangyan/sql-ai/v3/make_dataset_rewrite.ipynb, 生成带有指定"改写指令"的分析过程
  - 用/opt/huangyan/sql-ai/v3/generate_dpo_data_from_manual.ipynb, 生成DPO负例数据 (忽视某一次"删除列"的改写指令)

发现一个问题: 

  - SQL优化的任务, 和DPO任务的形式不一样. 
  - 所以将验证改成二段验证: 先让SQL优化任务输出分析过程, 然后用SQL+分析过程让DPO后的模型输出SQL结果.
    - 验证结果正确, 指令得到了遵循
  - 去除"删除列的指令没有被遵守"的DPO补充数据, 指令仍然能得到遵循
  - 下一步: 修正评估脚本, 完全二阶段验证, 进行端到端的验收

使用二阶段验证的评估结果: [1307.txt](/assets/01KJBZSCARJVG6RJPDTR56Z3C2/1307.txt)

commit: de8bac6e3e5c9265a9f1975bdda647d6e18f21bc

checkpiont另存: /opt/huangyan/sql-ai/checkpoints/v95-20250414-150059

评估效果非常好, SQL的改写指令能完全被遵守. 

下一步就回到SQL优化任务, 生成更好的分析过程即可

# 分析之前优化的结果

```
 SELECT AVG(A44_T_1_.fd_roi) AS "T_A81_2_" FROM (
     SELECT v1.months, t1.fd_ggr / v3.fd AS fd_roi
     FROM (
         SELECT SUBSTRING(pt, 1, 6) AS months
         FROM dwd.dwd_order_detail_basic_ri
         GROUP BY SUBSTRING(pt, 1, 6)
     ) v1
     LEFT JOIN (
         SELECT SUM(bingoggr) AS fd_ggr, SUBSTRING(pt, 1, 6) AS months
         FROM dwd.dwd_order_detail_basic_ri
         WHERE login_name IN (
             SELECT login_name FROM dwd.dwd_user_info_ri WHERE substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) =substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )
         )
         AND substring(TO_CHAR(current_date::timestamp, 'YYYYMMDD'),1,6 )=substring(TO_CHAR(pt::timestamp, 'YYYYMMDD'),1,6 )
         GROUP BY SUBSTRING(pt, 1, 6)
     ) t1 ON v1.months = t1.months
     LEFT JOIN (
         SELECT sum(cost) AS cost, state_date AS state_date
         FROM (
             SELECT sum(cost) AS cost, state_date FROM dwd.dwd_market_channel_cost_di WHERE state_cycle='MONTH' GROUP BY state_date
             UNION ALL
             SELECT sum(cost) AS cost, SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6) AS state_date FROM dwd.dwd_market_channel_cost_di WHERE state_cycle='DAY' GROUP BY
             SUBSTRING( TO_CHAR(state_date::timestamp, 'YYYYMMDD'),1,6)
         ) a
         GROUP BY a.state_date
     ) v2 ON v1.months = v2.state_date
     LEFT JOIN (
         SELECT substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 ) AS months, count(1) AS fd
         FROM dwd.dwd_user_info_ri
         GROUP BY substring(TO_CHAR(first_deposit_date::timestamp, 'YYYYMMDD'),1,6 )
     ) v3 ON v1.months = v3.months
 ) AS A44_T_1_ LIMIT 1 OFFSET 0
``` 

两个问题: 

  - v1.months的引用没有被删除, 其特征是LEFT JOIN的最左表
  - v2表没有被删除, 是因为之前的规则要求不能删除, 需要新的规则
