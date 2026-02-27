---
title: 20250813 - 继续进行SQL优化的GRPO训练
confluence_page_id: 4161998
created_at: 2025-08-13T14:22:46+00:00
updated_at: 2025-08-21T14:31:49+00:00
---

# 前继

[20250801 - 进行SQL优化的GRPO - 缩小训练效果和校验效果的差异] 中

  - 尝试11中, 对训练和评估的reward崩溃阶段进行了分析, 但还没有修改

由于训练时间越来越长, 所以引入了A100的机器进行对比测试, 对其时间/内存占用进行了分析: 

  - [20250811 - 诊断GRPO训练过程的时间/内存消耗]
  - 结论: 
    - 修正一个trl问题, 可减少内存尖峰
    - 关闭log_entropy, 可减少内存尖峰
    - prompt和completion太长, 会同时拉大torch和vllm的内存使用, 这是主要因素

# 需要减少completion长度, 来换取训练内存和时间

预计将推理过程转换成函数形式, 等将训练数据修复完, 再修复reward函数, 并修复之前"尝试11"中发现的数据崩溃的问题

但检查了尝试11的训练的输出, 发现length并不长: 

![image2025-8-13 23:52:53.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-13%2023%3A52%3A53.png)

只有很少的情况出现length=11000的情况: 

![image2025-8-13 23:54:5.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-13%2023%3A54%3A5.png)

考虑可以直接缩小model length相关参数到6000, 而不改变训练数据

现在可以修复尝试11中发现的训练/评估的奖励崩溃的问题

# 修复尝试11中发现的训练/评估的奖励崩溃的问题

  1. 在reward中, 支持多列模式, 举例: 
     1. ee

```
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(order_weight, create_time, delivery_status, transport_mode, distance_km, estimated_hours, customer_id, order_volume, payment_method): 没有被同级或父级语句(主语句-1)引用, 可以删除。
``` 
  2. 在reward中, 支持"可以删除"后, 出现其他分析内容, 举例: 

```
- 动作3: 检查语句(子查询inner_activity-1.1.1)的查询字段(activity_type): (检查结果), 可以删除。因为引用此字段的父查询(主FROM子语句-1.1)中的activity_type字段将被删除, 所以此字段也变得无用。
```

  3. 在reward中, 对于as的匹配逻辑进行升级: 

```
判断两个查询字段是否匹配
        满足以下条件之一即可认为是匹配的：
        1. 如果两个字段都包含 "as"，那么必须as后的部分匹配
        2. 如果只有一个字段包含as，那么另一个字段只需要精确匹配前半部分或者后半部分即可
        3. 如果不包含as，那么必须精确匹配
```

  4. reward需要支持"。可以删除", "。因此可以删除" , "，可删除", ", 所以可以删除","(主语句-1未使用该字段，可删除)", "**未被使用, 可以删除**"

还未修复: 

  - 对于UNION ALL, GT中值对UNION涉及的子句进行了判断, 而模型输出中, 先对UNION句子本身进行了判断, 然后才对两个子句进行了判断, 所以判断数量多于GT
  - GT中, 对UNION ALL的判断, 只判断了父语句, 而推理中判断了父语句+子语句
  - 对UNION的判断, 会遗漏一些字段的分析, 需要在reward中进行检查?

# 尝试12

减少模型生成长度为6000:

```
--max_length 2000 \
--max_completion_length 6000 \
--vllm_max_model_len 8000 \
``` 

出现了奇怪的现象: eval的F1 reward始终为0附近

![image2025-8-14 22:39:42.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-14%2022%3A39%3A42.png)

检查eval completion, 发现表名出错 (应为"子查询v1-1.1.1", 模型给出了"v1-1.1.1"): 

![image2025-8-14 22:42:39.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-14%2022%3A42%3A39.png)

将之前做得一次训练(正确reward) 和 当前训练(错误reward)进行对比:

  - 对sft的结果进行对比, sft的结果差不多. 命令如下: 

```
CUDA_VISIBLE_DEVICES=0 swift infer     --adapters /data/huangyan/sql-ai-2/train_logs/2025-08-08_13-03-17/train_output/v0-20250808-130506/checkpoint-84   --infer_backend pt     --temperature 0.6     --max_new_tokens 6000  --val_dataset /data/huangyan/sql-ai-2/train_logs/2025-08-08_13-03-17/val_data.jsonl    --max_batch_size 1 

CUDA_VISIBLE_DEVICES=0 swift infer     --adapters /data/huangyan/sql-ai-2/train_logs/2025-08-14_19-54-33/train_output/v0-20250814-195604/checkpoint-84  --infer_backend pt     --temperature 0.6     --max_new_tokens 6000  --val_dataset /data/huangyan/sql-ai-2/train_logs/2025-08-08_13-03-17/val_data.jsonl    --max_batch_size 1 
```

  - 检查GRPO的结果, 对于当前训练(错误reward)的初期和末期的checkpoint进行 infer, 发现生成的也没有问题. 但注意到: 

```
{'num_prompt_tokens': 3217, 'num_generated_tokens': 2079, 'num_samples': 1, 'runtime': 112.8966078623198, 'samples/s': 0.008857662058540543, 'tokens/s': 18.415079419705787}
```

    - 当前配置为 --max_length 2000 (GRPO中, 相当于max_prompt_length), 该值不合理
    - 使用参数: 

```
--max_length 4000 \
--max_completion_length 6000 \
--vllm_max_model_len 10000 \

```

      - 得到结果: 

```
{'eval_loss': 0.00201266, 'eval_completions/mean_length': 3043.625, 'eval_completions/min_length': 1910.0, 'eval_completions/max_length': 4083.0, 'eval_completions/clipped_ratio': 0.0, 'eval_rewards/Action3F1Reward/mean': 0.17297009, 'eval_rewards/Action3F1Reward/std': 0.27072686, 'eval_rewards/CompletenessReward/mean': 0.2, 'eval_rewards/CompletenessReward/std': 0.0, 'eval_rewards/ActionCountReward/mean': 0.12362607, 'eval_rewards/ActionCountReward/std': 0.0705989, 'eval_rewards/FormatErrorReward/mean': 0.0, 'eval_rewards/FormatErrorReward/std': 0.0, 'eval_reward': 0.49659616, 'eval_reward_std': 0.23564558, 'eval_frac_reward_zero_std': 0.0, 'eval_kl': 0.050338, 'eval_clip_ratio/low_mean': 0.0, 'eval_clip_ratio/low_min': 0.0, 'eval_clip_ratio/high_mean': 0.0, 'eval_clip_ratio/high_max': 0.0, 'eval_clip_ratio/region_mean': 0.0, 'eval_runtime': 59.5554, 'eval_samples_per_second': 0.017, 'eval_steps_per_second': 0.017, 'epoch': 0.36, 'global_step/max_steps': '10/560', 'percentage': '1.79%', 'elapsed_time': '15m 49s', 'remaining_time': '14h 30m 5s'}
```

    - 使用参数: 

```
--max_length 10000 \
--max_completion_length 10000 \
--vllm_max_model_len 10000 \

```

      - 得到结果: 

```
{"eval_loss": 0.00217436, "eval_completions/mean_length": 3264.0, "eval_completions/min_length": 2278.0, "eval_completions/max_length": 4316.0, "eval_completions/clipped_ratio": 0.0, "eval_rewards/Action3F1Reward/mean": 0.52726722, "eval_rewards/Action3F1Reward/std": 0.30220807, "eval_rewards/CompletenessReward/mean": 0.2, "eval_rewards/CompletenessReward/std": 0.0, "eval_rewards/ActionCountReward/mean": 0.12772119, "eval_rewards/ActionCountReward/std": 0.05910047, "eval_rewards/FormatErrorReward/mean": 0.0, "eval_rewards/FormatErrorReward/std": 0.0, "eval_reward": 0.85498834, "eval_reward_std": 0.34915462, "eval_frac_reward_zero_std": 0.0, "eval_kl": 0.05440409, "eval_clip_ratio/low_mean": 0.0, "eval_clip_ratio/low_min": 0.0, "eval_clip_ratio/high_mean": 0.0, "eval_clip_ratio/high_max": 0.0, "eval_clip_ratio/region_mean": 0.0, "eval_runtime": 63.5616, "eval_samples_per_second": 0.016, "eval_steps_per_second": 0.016, "epoch": 0.35714286, "global_step/max_steps": "10/560", "percentage": "1.79%", "elapsed_time": "16m 3s", "remaining_time": "14h 43m 26s"}
```

# 尝试13 - 修复了生成长度问题

修复了: 

  - 修复尝试11中发现的训练/评估的奖励崩溃的问题
  - 修复尝试12中的生成长度问题

当前状态: "epoch": 13.21428571, "global_step/max_steps": "370/560", "percentage": "66.07%", "elapsed_time": "10h 13m 58s", "remaining_time": "5h 15m 17s"

与修复前的结果相比: (深蓝色为修复前, 浅蓝色为修复后)

![image2025-8-15 10:14:55.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-15%2010%3A14%3A55.png)

  - eval/Action3F1 达到新高
  - eval/ActionCount 比之前上升趋势低, 需要考虑原因
  - train的三项reward都有效率提升
  - train/ActionCount都会经历一个下降趋势, 需要考虑原因
  - 对completion进行分析: 
    - 1/2 这2项数据 达不到 1.0, 需要分析
      - case 1: 模型无法区分别名相似的两张表的列引用的区别: "- 动作3: 检查语句(JOIN子查询-1.1)的查询字段(zz1.zuowu_leixing): 被同级或父级语句(主语句-1)作为(选择字段)引用: (zz.zuowu_leixing)"
        - ![image2025-8-15 10:37:48.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-15%2010%3A37%3A48.png)
      - case 2: 与UNION相关, 之前分析过
    - 7/8/18/34/48/58 这6个数据, 在训练中的趋势都偏离了1.0, 需要分析
      - case 7: 模型混淆了两张表的相同列名的引用分析 (之前是正确的, 随着训练进行, 混淆加深)
        - ![image2025-8-15 10:44:1.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-15%2010%3A44%3A1.png)
      - case 8: 与UNION相关, 之前分析过
      - case 18: 与UNION相关, 之前分析过
      - case 34:
        - "检查语句(主JOIN子查询-1.1)的查询字段(CONCAT(YEAR(sc.created_at), '-', MONTH(sc.created_at)) AS connection_period)"
        - 函数列带有逗号, 与 多字段 的解析冲突, 需要修正reward函数
      - case 48:
        - 模型混淆了两张表的相同列名的引用分析, 与case 7类似: 
        - ![image2025-8-15 10:59:31.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-15%2010%3A59%3A31.png)
      - case 58 (评估数据):
        - 出现了一些未被使用的变种描述: 

```
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.bingoggr): 被同级或父级语句(主语句-1)作为(计算列)引用: (fd_roi中使用了t1.fd_ggr和v3.fd, v1.bingoggr没有被使用)
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v2.cost): 被同级或父级语句(主语句-1)作为(计算列)引用: (主语句-1中没有用到cost字段)
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v2.cost): 被同级或父级语句(主语句-1)作为(选择字段/表连接列/计算列/...)引用: (v2.cost 并未在最终选择中被使用)
- 动作3: 检查语句(子查询v1-1.1.1)的查询字段(valid_account): 被同级或父级语句(主FROM子语句-1.1)作为(未引用)引用: (主FROM子语句-1.1中没有使用 valid_account)
- 动作3: 检查语句(主FROM子语句-1.1)的查询字段(v1.valid_account): 被同级或父级语句(主语句-1)作为(选择字段/表连接列/计算列)引用: (未被使用)
```

        - 在训练的中间阶段 (会维持一段低分段), 出现逐个语句检查时, 遗漏了一些语句的检查: 
          - ![image2025-8-15 11:23:2.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-15%2011%3A23%3A2.png)

最终的训练结果: ("global_step/max_steps": "515/560", "percentage": "91.96%", "elapsed_time": "15h 4m 2s", "remaining_time": "1h 18m 59s")

  - train/Action3F1Reward 已经到0.99, eval/Action3F1Reward 已经到0.85

![image2025-8-15 15:8:56.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-15%2015%3A8%3A56.png)

# 修正数据

  - 修正reward问题: "函数列带有逗号, 与 多字段 的解析冲突, 需要修正reward函数"
  - 针对"在训练的中间阶段 (会维持一段低分段), 出现逐个语句检查时, 遗漏了一些语句的检查", 增加新的reward: sql_ai_action1_statement_coverage
  - 统一对UNION (ALL) 的处理, 单独重新生成union相关的数据
  - 在FormatErrorReward中, 检查"动作3: 检查语句(...)的查询字段(...): 被同级或父级语句(...)作为(...)引用: (...)"中, 最后两个括号中, 不应出现"没有使用"这样的否定字样 (应当使用动作3的另外一种形式)

没有进行改进: 

  - "模型混淆了两张表的相同列名的引用分析"

# 尝试14

修正数据, 并改用zero2

![image2025-8-15 15:44:3.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-15%2015%3A44%3A3.png)

eval 起手变低

在改回zero3进行对比: (绿色为zero2, 橙色为zero3), 起手略好于zero2, 原因不明

![image2025-8-15 16:29:1.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-15%2016%3A29%3A1.png)

使用zero3训练, commit: 993167eeab2d79be105f18033a409481df38ff10

  - 训练时间: "epoch": 20.0, "global_step/max_steps": "560/560", "percentage": "100.00%", "elapsed_time": "16h 17m 17s", "remaining_time": "0s"
  - ![image2025-8-16 20:39:47.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-16%2020%3A39%3A47.png)
  - train阶段:
    - reward在 step=350 附近达到最高, 达到理论极值
  - eval阶段  

    - eval在step=400 附近达到最高, Action3F1在0.84左右
    - eval的ActionCount很低, 需要检查
      - 经过检查: GT中, "优化后的检查" 步骤很短 (动作比较少), 而train数据教会模型应当详细进行 "优化后的检查" , 所以动作步数会比GT高. 这一趋势是正确的, 应当修正GT
  - 可以认为是 train数据 已经不足 推理出eval数据了
  - 检查completion
    - prompt 1没有达到过极值
    - prompt 24是eval数据, 长期达不到极值
      - 检查数据格式等没有什么问题, 先增加更多数据来训练, 如果仍不得提高, 需要检查SQL语义等劣化趋势

使用zero2训练 做一个对比: 训练趋势没差别, 训练时间: (使用相同的epoch, zero2 会快10%)

  - zero2: "epoch": 15.35714286, "global_step/max_steps": "430/560", "percentage": "76.79%", "elapsed_time": "10h 53m 25s", "remaining_time": "3h 17m 32s"
  - zero3: "epoch": 15.35714286, "global_step/max_steps": "430/560", "percentage": "76.79%", "elapsed_time": "12h 6m 28s", "remaining_time": "3h 39m 38s"

![image2025-8-16 22:0:59.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-16%2022%3A0%3A59.png)

# 尝试15 - 增加训练数据

当前使用56个数据, 增加训练数据到75个, 训练效果: 

(白色是本次实验, 红色是56个数据的实验)

(耗时: "epoch": 20.0, "global_step/max_steps": "720/720", "percentage": "100.00%", "elapsed_time": "18h 52m 18s", "remaining_time": "0s")

![image2025-8-17 23:12:59.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-17%2023%3A12%3A59.png)

注意: 本轮重新生成过评估数据

发现: 

  - train/reward 收敛变慢, 在结束时才接近理论极值. ActionCount/Action3F1/Completeness 都会有比较大的偏离
  - eval的各项reward在 step=400后都下降比较厉害, 且各项reward都不高
  - 检查completion:
    - prompt 1无法达到极值, 接近于0
      - 判断为GT错误, 对UNION的处理, 推理是正确的, GT是错误的, 手工删除该数据
        - 数据线索: "select a.yonghu_ming, a.dengji, c.neirong, c.fabu_shijian from yonghu a join ..."
    - prompt 22 是评估数据, 比较低
      - 从step=420左右, 开始直接跳过外层查询, 从v1/v2表开始诊断, 导致诊断结果错误 (需要删除最外层的选择列, 才能处理内层的)
        - 在 step=260左右, 就有这个趋势, 但后面得到了修正
    - prompt 28/41/45 大部分不够接近极值
      - prompt 28, 混淆了不同表的同名列:
        - ![image2025-8-17 23:35:49.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-17%2023%3A35%3A49.png)
      - prompt 41, 混淆了不同表的同名列:
        - ![image2025-8-17 23:41:59.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-17%2023%3A41%3A59.png)
      - prompt 45, 混淆了不同表的同名列:
        - ![image2025-8-17 23:48:0.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-17%2023%3A48%3A0.png)
  - 需要检查训练数据的各项reward, 尤其是Statement coverage, 不知道eval数据是从哪里学到了跳过外层查询
    - Statement coverage 只检查 动作1和动作2/3之间的差异, 但在eval数据的错误中, 动作1也直接跳过了外层查询. 需要设计其他reward来处理? 
  - 需要检查训练数据中, 是否有混淆不同表的同名列的问题

先跑一轮删除了错误数据(1条)的训练:

(蓝色是最新训练, 灰色是删除错误数据前的训练)

![image2025-8-18 10:3:6.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-18%2010%3A3%3A6.png)

效果: 

  - train阶段 收敛更快 (一条错误数据就造成了很大影响)
  - eval没有太大改善
  - 检查completion
    - 2个数据不能达到极值
      - prompt 1是评估数据
        - 与上一轮一样, 跳过了外层查询的分析, 导致错误
      - prompt 2
        - GT错误, 出现了"因此可以从SELECT列表中删除。"这种reward无法解析的描述. 手工修复
    - 长期不能达到极值  

      - prompt 41
        - 出现俄文, 认为是因为训练没结束
    - 劣化
      - prompt 22
        - 字段没列举完: 
          - ![image2025-8-18 10:25:58.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-18%2010%3A25%3A58.png)
        - 出现饿文和英文
      - prompt 23
        - 出现乱码
      - prompt 32
        - 某一列在外层被作为条件使用, 但训练认为需要删除它. 先修正提示词?
        - ![image2025-8-18 10:36:27.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-18%2010%3A36%3A27.png)
      - prompt 52
        - 对列是否在父查询中引用, 进行了错误的判断, 原因不详
      - prompt 63
        - 中间出现日文/俄文等

# 尝试16 - 对训练数据进行reward检查

使用脚本 /data/huangyan/sql-ai-2/grpo/check_reward_functions_on_train_data.py 进行reward检查, 得到结果: 

![image2025-8-18 11:13:10.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-18%2011%3A13%3A10.png)

有不少case在statement coverage上有扣分

进行变更: 

  - 在prompt中, 增加提示, 需要满足statement coverage的要求. 
  - 在生成数据时增加对reward函数的检测, 要求不能扣分.
  - 增加要求: 回答要使用中文
  - 解决"某一列在外层被作为条件使用, 但训练认为需要删除它.

重新生成所有数据, 使用73条数据进行训练: 

![image2025-8-18 16:5:16.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-18%2016%3A5%3A16.png)

训练直接崩. 分析completion: 下降阶段都出现了 乱码(俄文/日文/等)

注意: 当前sft的LR是5e-5, 但GRPO的LR是1e-4, 降到1e-5试一下

# 检查 "混淆了不同表的同名列" 

使用脚本: /data/huangyan/sql-ai-2/make_train_dataset/check_sql_analysis_on_column_mistaken.py

检查训练数据中是否存在混淆问题, 经过deepseek-v3检查, 训练数据不存在问题. 

提高到deepseek-r1来进行检查, 检查出两处错误: 

  1. 数据确实是出现了错误, 需要删除:

```
SELECT a.booking_id, a.user_id, a.trip_id, a.departure_date, a.return_date FROM t_trip_booking a JOIN (SELECT b.booking_id, b.user_id, b.trip_id, b.departure_date, b.return_date, b.booking_status, b.total_price, b.payment_method, b.discount_code, b.created_at, b.updated_at FROM t_trip_booking b WHERE b.departure_date >= NOW()) c ON a.booking_id = c.booking_id WHERE a.user_id = 123;
```

  2. 产生了如下输出, 导致了误解:

```
- 动作3: 检查语句(子查询-1.1.1)的查询字段(soil_type): 被同级或父级语句(主FROM子语句-1.1)作为(选择字段, ORDER BY排序列)引用: 无
```

手工修复以上两个问题, 再进行训练 (实际删除了第一个数据, 第二个数据因为其他过滤, 并没有进入训练)

注意: 因为问题数量少, 并没有加到"数据生成"的过程

# 尝试17 - 调低GRPO LR=1e-5, 修复以上两个数据问题

训练效果: 

![image2025-8-19 10:4:34.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-19%2010%3A4%3A34.png)

  - 训练时间: "epoch": 18.05555556, "global_step/max_steps": "650/720", "percentage": "90.28%", "elapsed_time": "17h 17m 53s", "remaining_time": "1h 51m 46s"
  - train
    - 在最后阶段, 各reward逐步接近极值
  - eval
    - Action3F1逐步下降, 与train的趋势不同
  - completion
    - eval数据: 从训练刚开始, 得分为0的推理大部分都是跳过了外层查询的分析 导致了错误. 但随着训练进行, 这个状况逐步严重. (训练方向上并没有改善这一项)
  - TODO:
    - 需要修复问题: "Statement coverage 只检查 动作1和动作2/3之间的差异. 但动作1会有时会跳过最外层的查询", 改进奖励函数

# 尝试18 - 修正statement coverage奖励

改进奖励函数, 要求 动作1 和 动作2/3 覆盖的语句名, 必须和SQL结构中所有的语句名 一致

改进奖励函数后, 发现大部分训练数据都造成了扣分: 

![image2025-8-19 11:29:28.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-19%2011%3A29%3A28.png)

重新构建训练数据: 76条

训练效果: 

  - 时间: "epoch": 12.08333333, "global_step/max_steps": "435/720", "percentage": "60.42%", "elapsed_time": "15h 8m 37s", "remaining_time": "9h 55m 18s"
  - 训练曲线: 浅蓝色是之前训练时崩溃的曲线, 深蓝色是本次训练的曲线  

    - ![image2025-8-20 10:13:8.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-20%2010%3A13%3A8.png)
  - 训练过程:   

    - 起点就已经接近reward极值, 认为是sft过拟合. 应当修正这种过拟合的状态
    - 在step=100附近, Action3F1和ActionCount有下降, 在completion分析中需要研究这个阶段
  - 评估过程:
    - Action3F1 从 0.46 上升到 0.84, 其他reward都保持的很好
    - 之前训练的评估崩溃, 被reward调整和训练数据筛选修正了
  - completion分析
    - 分析训练过程中: 在step=100附近, Action3F1和ActionCount有下降
    - TODO

# 尝试19 - 在GRPO阶段中, 混入部分sft中没见过的数据. LR放大一倍

sft阶段: 76条

GRPO阶段: 96条

将LR调整成2e-5 (之前的训练太稳了)

训练效果: 

  - 时间:"epoch": 10.10416667, "global_step/max_steps": "485/960", "percentage": "50.52%", "elapsed_time": "14h 52m 22s", "remaining_time": "14h 33m 58s"
  - 训练曲线: 
    - ![image2025-8-21 1:37:8.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-21%201%3A37%3A8.png)
    - 浅蓝色是本次训练, 深蓝色是加入陌生数据之前的训练. 
    - 不管从train还是eval看来, reward的极值都达不到 上一次训练 的程度 (但本次训练只进行到一半)
    - 分析completion
      - prompt 1-7, reward不到极值
        - prompt 1: case的SQL错误: 目前可以手工删除
          - union 左右字段数不匹配

```
SELECT patient_id, COUNT(*) AS visit_count FROM ( SELECT pv.patient_id, pv.visit_date FROM ( SELECT pd.patient_id, pd.patient_name, pv.visit_date FROM t_patient_details pd INNER JOIN t_patient_visits pv ON pd.patient_id = pv.patient_id WHERE pd.gender = 'F' ) AS subquery1 UNION ALL SELECT mp.patient_id, mp.prescription_date AS visit_date FROM t_medication_prescriptions mp WHERE mp.medication_name = 'Paracetamol' ) AS subquery2 GROUP BY patient_id;
``` 
        - prompt 2: GT中出现了"故可以删除"字样, reward函数需要支持
        - prompt 3: GT错误, 可以手工删除数据: 

```
select shebei_mingcheng, weihuo_time from ( select shebei_mingcheng, weihuo_time, chuangjian_time from ( select shebei_mingcheng, weihuo_time, chuangjian_time, shebei_zhuangtai from ( select shebei_bianhao, shebei_mingcheng, weihuo_time, chuangjian_time, shebei_zhuangtai from shebei where shebei_zhuangtai = 1 order by chuangjian_time desc ) t1 where shebei_zhuangtai = 1 order by chuangjian_time desc ) t2 where shebei_zhuangtai = 1 order by chuangjian_time desc ) t3 order by weihuo_time desc;
```

        - prompt 4: 包含级联删除, 因为sft可能没有类似数据, 所以不能识别

```
select j.xingming, y.xingming as yisheng_xingming from (select id, xingming, shoujihao from (select id, xingming, shoujihao, substring(wenti, 1, 50) as wenti_jianyao from (select id, xingming, shoujihao, wenti from jiankangzixun where riqi > '2023-01-01') as t1) as t2) as j join (select id, xingming from yisheng where zhuanye = 'neike') as y on j.id in (select jiankangzixun_id from huifu where yisheng_id = y.id);
```

          - wenti_jianyao 没有被使用, wenti也可以被删除, 但会识别成wenti被使用

        - prompt 5: GT中, 出现了以"/"分割的多字段, 先删除数据

```
select chepaihao, xingshilicheng from (select chepaihao, xingshilicheng, youhao from cheliangxinxi where chucheshijian > '2023-01-01' union all select chepaihao, null as xingshilicheng, feiyong from cheliangweixiu where weixiushijian > '2023-01-01') as temp order by xingshilicheng;
```

        - prompt 6: GT中, 对于UNION的父语句, 出现了如下描述: 

```
  - 动作2: 检查语句(主FROM子语句-1.1)的各查询字段: (hotel_id, user_id, booking_date)
    - 动作3: 根据规则说明 "对于集合操作类SQL（UNION、UNION ALL、INTERSECT、EXCEPT等），跳过集合操作的父语句，直接检查各个子查询", 因此直接检查其子语句 `UNION第一部分-1.1.1` 和 `UNION第二部分-1.1.2`。
```

        - prompt 7: 评估数据
          - 格式没有问题
      - prompt 13, 长期不到极值
        - GT在Union父语句中, 出现了如下描述, 而模型并未学会这种描述: 

```
- 动作2: 检查语句(JOIN子查询-1.1)的各查询字段: (该语句为UNION ALL集合操作, 根据规则描述跳过父语句, 直接检查其子查询`UNION第一部分-1.1.1`和`UNION第二部分-1.1.2`)
```

      - prompt 24, reward下降
        - 模型混淆了 不同表的同名列:
          - ![image2025-8-21 13:1:48.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-21%2013%3A1%3A48.png)  
  

# 尝试20 - 变更GSPO的参数

依据: 

![image2025-8-19 10:9:33.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-19%2010%3A9%3A33.png)

参数: 

```
从
    --epsilon 0.2 \
    --epsilon_high 0.28 \
    --beta 0.04 \

变更为
    --epsilon 3e-4 \
    --epsilon_high 4e-4 \
    --beta 0 \

``` 

整体训练时间变长?

```
从
{"loss": -0.02606414, "grad_norm": 0.03925246, "learning_rate": 1.789e-05, "memory(GiB)": 52.21, "train_speed(iter/s)": 0.010115, "kl": 0.01980286, "clip_ratio/low_mean": 0.0, "clip_ratio/low_min": 0.0, "clip_ratio/high_mean": 0.0, "clip_ratio/high_max": 0.0, "clip_ratio/region_mean": 0.0, "completions/mean_length": 2873.40625, "completions/min_length": 1766.0, "completions/max_length": 6567.0, "completions/clipped_ratio": 0.03125, "rewards/Action3F1Reward/mean": 0.94699323, "rewards/Action3F1Reward/std": 0.07072374, "rewards/CompletenessReward/mean": 0.2, "rewards/CompletenessReward/std": 0.0, "rewards/ActionCountReward/mean": 0.19965312, "rewards/ActionCountReward/std": 0.00277509, "rewards/FormatErrorReward/mean": 0.0, "rewards/FormatErrorReward/std": 0.0, "rewards/Action1StatementCoverageReward/mean": 0.0, "rewards/Action1StatementCoverageReward/std": 0.0, "reward": 1.34664655, "reward_std": 0.05357206, "frac_reward_zero_std": 0.0, "epoch": 5.0, "global_step/max_steps": "240/960", "percentage": "25.00%", "elapsed_time": "6h 34m 20s", "remaining_time": "19h 43m 2s"}

 
变成了
 
{"eval_loss": -3.7e-07, "eval_completions/mean_length": 4927.5, "eval_completions/min_length": 4335.0, "eval_completions/max_length": 5652.0, "eval_completions/clipped_ratio": 0.0, "eval_rewards/Action3F1Reward/mean": 0.6705116, "eval_rewards/Action3F1Reward/std": 0.17901109, "eval_rewards/CompletenessReward/mean": 0.2, "eval_rewards/CompletenessReward/std": 0.0, "eval_rewards/ActionCountReward/mean": 0.2, "eval_rewards/ActionCountReward/std": 0.0, "eval_rewards/FormatErrorReward/mean": 0.0, "eval_rewards/FormatErrorReward/std": 0.0, "eval_rewards/Action1StatementCoverageReward/mean": 0.0, "eval_rewards/Action1StatementCoverageReward/std": 0.0, "eval_reward": 1.07051158, "eval_reward_std": 0.17901114, "eval_frac_reward_zero_std": 0.0, "eval_clip_ratio/low_mean": 0.0, "eval_clip_ratio/low_min": 0.0, "eval_clip_ratio/high_mean": 0.0, "eval_clip_ratio/high_max": 0.0, "eval_clip_ratio/region_mean": 0.0, "eval_runtime": 75.6851, "eval_samples_per_second": 0.013, "eval_steps_per_second": 0.013, "epoch": 5.0, "global_step/max_steps": "240/960", "percentage": "25.00%", "elapsed_time": "8h 16m 49s", "remaining_time": "1d 0h 50m 28s"}
``` 

曲线对比: reward趋势 会比 调整前 更好

![image2025-8-21 10:40:58.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-21%2010%3A40%3A58.png)

但为什么时间长度会增加: 

  - prepare_inputs的时间统计, beta=0(紫色曲线) 会比 beta>0 的时间稍短
    - ![image2025-8-21 10:46:30.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-21%2010%3A46%3A30.png)
  - 但在generate中, 紫色曲线有一个很大的偏离:
    - ![image2025-8-21 10:47:46.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-21%2010%3A47%3A46.png)

# 尝试21 - 调整尝试19的数据 (A100)

解决问题: 

  - 删除了三组有错误的数据
  - 在reward中, 增加"故可以删除"字样的支持
  - 在提示词中规范 UNION的父语句 的描述 ("跳过集合操作的父语句，直接检查各个子查询")

暂不解决的问题: 

  - 模型混淆了 不同表的同名列
  - 包含级联删除, 因为sft可能没有类似数据, 所以不能识别

切换到A100上执行, 调整参数: 去除deepspeed

![image2025-8-21 22:25:47.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-21%2022%3A25%3A47.png)

eval崩溃, 不知是否是deepspeed参数的调整问题

# 尝试22 - GRPO不使用额外的数据, 而是让sft早停 (4090)

将sft的epoch从12调整到6

GRPO和sft使用相同的数据

GRPO的LR调整为 4e-5

蓝色曲线是尝试19, 橙色曲线是当前实验. 等待训练完成

![image2025-8-21 22:30:32.png](/assets/01KJBZY39GJE38CB7BYSTRWMBX/image2025-8-21%2022%3A30%3A32.png)

# 待解决的问题

  - ~~混淆了不同表的同名列~~
  - ~~Statement coverage 只检查 动作1和动作2/3之间的差异. 但动作1会有时会跳过最外层的查询. 需要观察影响.~~
  - 在语句检查时, 字段没列举完 (动作2和其下的动作3之间不匹配). 需要观察影响.
  - ~~在GRPO阶段, 引入sft阶段没见过的陌生数据~~
  - sft和GRPO迭代使用, 减少GRPO的时长
  - ~~让sft阶段的训练epoch减少 (早停), 增加GRPO阶段的不确定性~~
