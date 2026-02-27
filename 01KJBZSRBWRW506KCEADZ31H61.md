---
title: 20250514 - 研究SQL优化模型微调 - 将需要多次迭代的优化合并到一次
confluence_page_id: 3997705
created_at: 2025-05-13T12:43:47+00:00
updated_at: 2025-05-21T02:48:51+00:00
---

# 额外 - 进行多次SQL劣化

(增加了"可以是你新建的列"的要求)

第一次劣化: 

```
我有一个SQL: 
----------
SELECT Name, Sales_in_Billion 
FROM (SELECT Name, ID, ID AS CID FROM company) AS c 
JOIN (SELECT Sales_in_Billion, ID FROM sales) AS s ON c.ID = s.ID
ORDER BY Sales_in_Billion DESC
----------

我需要你将其按照如下要求进行改写:
- 谨遵以下要求, 修改不要超过要求, 且只修改一处, 且不要增加注释说明
- 不要在一个位置上反复修改. 鼓励你在不同的位置上做修改
- 改写后的SQL应当是能正确运行的, 不能因为改写导致SQL格式错误或计算错误或其他错误

要求: 
----------
在子语句中增加 结果 字段 (命名需要符合业务意义), 增加的 结果字段 (可以是你新建的列) 不能改变SQL的最终结果, 如果删除这个 结果字段, 也不能改变SQL的最终结果
----------

输出改写后的SQL

SELECT Name, Sales_in_Billion 
FROM (SELECT Name, ID, ID AS CID, 'Active' AS Status FROM company) AS c 
JOIN (SELECT Sales_in_Billion, ID FROM sales) AS s ON c.ID = s.ID
ORDER BY Sales_in_Billion DESC
``` 

第二次劣化: 

```
我有一个SQL: 
----------
SELECT Name, Sales_in_Billion 
FROM (SELECT Name, ID, ID AS CID, 'Active' AS Status FROM company) AS c 
JOIN (SELECT Sales_in_Billion, ID FROM sales) AS s ON c.ID = s.ID
ORDER BY Sales_in_Billion DESC
----------

我需要你将其按照如下要求进行改写:
- 谨遵以下要求, 修改不要超过要求, 且只修改一处, 且不要增加注释说明
- 不要在一个位置上反复修改. 鼓励你在不同的位置上做修改
- 改写后的SQL应当是能正确运行的, 不能因为改写导致SQL格式错误或计算错误或其他错误

要求: 
----------
在子语句中增加 结果 字段 (命名需要符合业务意义), 增加的 结果字段 (可以是你新建的列) 不能改变SQL的最终结果, 如果删除这个 结果字段, 也不能改变SQL的最终结果
----------

输出改写后的SQL

SELECT Name, Sales_in_Billion 
FROM (SELECT Name, ID, ID AS CID, 'Active' AS Status FROM company) AS c 
JOIN (SELECT Sales_in_Billion, ID, 1 AS Sales_Rank FROM sales) AS s ON c.ID = s.ID
ORDER BY Sales_in_Billion DESC
``` 

# 尝试的方案

重新生成rule-1的训练数据, 训练, 并在此基础上进行GRPO

# 训练1

使用的基模型: (对应[20250514 - 研究SQL优化模型微调 - 将rule-1改写成伪代码模式]中的训练2)

  - /opt/huangyan/sql-ai/output/Qwen3-8B/v17-20250514-150148/checkpoint-192

commit: 7ecc842a475e3e11e149638006f455b010ff12b2

使用GRPO的参数: 

```
  bash -c "CUDA_VISIBLE_DEVICES=0,1,2,3 PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True \
  NPROC_PER_NODE=1 \
  swift rlhf \
    --rlhf_type grpo \
    --train_type lora \
    --torch_dtype bfloat16 \
    --per_device_train_batch_size 2 \
    --per_device_eval_batch_size 2 \
    --learning_rate 1e-6 \
    --lora_rank 8 \
    --lora_alpha 32 \
    --gradient_accumulation_steps 1 \
    --eval_steps 10 \
    --save_steps 10 \
    --save_total_limit 5 \
    --logging_steps 5 \
    --warmup_ratio 0.05 \
    --use_vllm false \
    --sleep_level 0 \
    --offload_optimizer true \
    --offload_model true \
    --gc_collect_after_offload true \
    --max_length 4096 \
    --max_completion_length 4096 \
    --num_generations 2 \
    --log_completions true \
    --ddp_find_unused_parameters false \
    $args \
    ${last_model_checkpoint:+--resume_only_model} \
    ${last_model_checkpoint:+--resume_from_checkpoint ${last_model_checkpoint}}" > "$log_file" 2>&1
``` 

在4个4090的GPU上运行, 将num_generations调小, 关闭了VLLM, 关闭了deepseek, NPROC_PER_NODE设置为1, 否则GPU显存不够用

使用的奖励函数: 验证输出是否为json (仅用于测试)

# 训练2

修订奖励函数: 

  - 如果某一层包括"改写前"和"改写后", 并且两者不同, 则加分

commit: 8937f7192084f36f375638fb54a09f2acbfe2876

训练结果: 

  - ![image2025-5-16 13:0:23.png](/assets/01KJBZSRBWRW506KCEADZ31H61/image2025-5-16%2013%3A0%3A23.png)
  - ![image2025-5-16 13:2:2.png](/assets/01KJBZSRBWRW506KCEADZ31H61/image2025-5-16%2013%3A2%3A2.png)

评论: 

  - 使用了sft相同的训练数据, 所以train/loss很低
  - 查看/opt/huangyan/sql-ai/output/Qwen3-8B/v75-20250515-232419/completions.jsonl, 其中的reward大部分都是0.1 (有1个改写), 也就是说在同一批中, 很难突破出不同的方向

# 训练3

使用与sft不同的数据? 让模型能更发散

用一些劣化两次的数据, 直接进行GRPO试试

测试1: 

  - 先用劣化一次的数据微调, 然后用劣化两次的数据(40条)进行微调
  - 最后用劣化两次的数据(50条)进行GRPO
  - GRPO时长: 13748 秒
  - train: 
    - ![image2025-5-19 13:35:51.png](/assets/01KJBZSRBWRW506KCEADZ31H61/image2025-5-19%2013%3A35%3A51.png)
    - ![image2025-5-19 13:39:22.png](/assets/01KJBZSRBWRW506KCEADZ31H61/image2025-5-19%2013%3A39%3A22.png)

测试2: 

  - 先用劣化一次的数据微调
  - 最后用劣化两次的数据(50条)进行GRPO
  - GRPO时长: 13343 秒
  - train:
    - ![image2025-5-19 13:40:18.png](/assets/01KJBZSRBWRW506KCEADZ31H61/image2025-5-19%2013%3A40%3A18.png)
    - ![image2025-5-19 13:40:43.png](/assets/01KJBZSRBWRW506KCEADZ31H61/image2025-5-19%2013%3A40%3A43.png)

好像没有效果. 

# 额外: 先找到是否能加速训练的方式

关掉以下offload选项: 

```
    --offload_optimizer false \
    --offload_model false \
    --gc_collect_after_offload false \
``` 

没有太大提升, 从10/245进度的预估, 从2:27:07减少到了2:14:24, 且不稳定

提高vllm_max_num_seqs: 

```
--vllm_max_num_seqs 512 \
``` 

性能下降

设置vllm_enforce_eager:

```
--vllm_enforce_eager true \
``` 

性能下降

设置deepspeed为zero1: 

性能下降

设置--dataloader_num_workers 4:

性能未见提升 (主要是为了节省tokenizer的时间)

使用colocate模式: 

```
--vllm_mode colocate \
``` 

没有找到能加速训练的方式

# 训练4

升级ms-swift到最新版本, 然后更新脚本

commit: 4e1cb751ce18a6efd07716f97027cf127a053bbf

基于sft模型, sft中仅使用一次劣化的数据

使用: CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2

num_generations=2, epoch=1

  - 中途报错: 

```
 [rank0]: Traceback (most recent call last):
[rank0]:   File "/opt/huangyan/ms-swift/swift/cli/rlhf.py", line 5, in <module>
[rank0]:     rlhf_main()
[rank0]:   File "/opt/huangyan/ms-swift/swift/llm/train/rlhf.py", line 173, in rlhf_main
[rank0]:     return SwiftRLHF(args).main()
[rank0]:   File "/opt/huangyan/ms-swift/swift/llm/base.py", line 47, in main
[rank0]:     result = self.run()
[rank0]:   File "/opt/huangyan/ms-swift/swift/llm/train/sft.py", line 151, in run
[rank0]:     return self.train(trainer)
[rank0]:   File "/opt/huangyan/ms-swift/swift/llm/train/sft.py", line 211, in train
[rank0]:     trainer.train(trainer.args.resume_from_checkpoint)
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/mixin.py", line 336, in train
[rank0]:     res = super().train(*args, **kwargs)
[rank0]:   File "/root/anaconda3/envs/sql-ai/lib/python3.10/site-packages/transformers/trainer.py", line 2245, in train
[rank0]:     return inner_training_loop(
[rank0]:   File "/root/anaconda3/envs/sql-ai/lib/python3.10/site-packages/transformers/trainer.py", line 2627, in _inner_training_loop
[rank0]:     self._maybe_log_save_evaluate(
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/mixin.py", line 394, in _maybe_log_save_evaluate
[rank0]:     super()._maybe_log_save_evaluate(tr_loss, *args, **kwargs)
[rank0]:   File "/root/anaconda3/envs/sql-ai/lib/python3.10/site-packages/transformers/trainer.py", line 3096, in _maybe_log_save_evaluate
[rank0]:     metrics = self._evaluate(trial, ignore_keys_for_eval)
[rank0]:   File "/root/anaconda3/envs/sql-ai/lib/python3.10/site-packages/transformers/trainer.py", line 3045, in _evaluate
[rank0]:     metrics = self.evaluate(ignore_keys=ignore_keys_for_eval)
[rank0]:   File "/root/anaconda3/envs/sql-ai/lib/python3.10/site-packages/transformers/trainer.py", line 4154, in evaluate
[rank0]:     output = eval_loop(
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 1138, in evaluation_loop
[rank0]:     output = super().evaluation_loop(dataloader, *args, **kwargs)
[rank0]:   File "/root/anaconda3/envs/sql-ai/lib/python3.10/site-packages/transformers/trainer.py", line 4348, in evaluation_loop
[rank0]:     losses, logits, labels = self.prediction_step(model, inputs, prediction_loss_only, ignore_keys=ignore_keys)
[rank0]:   File "/root/anaconda3/envs/sql-ai/lib/python3.10/site-packages/trl/trainer/grpo_trainer.py", line 1255, in prediction_step
[rank0]:     inputs = self._prepare_inputs(inputs)
[rank0]:   File "/root/anaconda3/envs/sql-ai/lib/python3.10/site-packages/trl/extras/profiling.py", line 87, in wrapper
[rank0]:     return func(self, *args, **kwargs)
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 321, in _prepare_inputs
[rank0]:     inputs = self._generate_and_score_completions(accumulated_local_batch)
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 811, in _generate_and_score_completions
[rank0]:     inputs = self._generate_completions(inputs)
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 786, in _generate_completions
[rank0]:     inputs, outputs = self._fast_infer(inputs)
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 762, in _fast_infer
[rank0]:     outputs = self._infer_single_or_multi_turn(inputs, self.request_config)
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 630, in _infer_single_or_multi_turn
[rank0]:     self._set_inputs_system(inputs)
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 608, in _set_inputs_system
[rank0]:     if all(_input['messages'][0]['role'] == 'system' for _input in inputs):
[rank0]: TypeError: 'NoneType' object is not iterable

``` 

num_generations=4, epoch=1

  - 总运行时间: 9445秒
  - ![image2025-5-19 23:12:28.png](/assets/01KJBZSRBWRW506KCEADZ31H61/image2025-5-19%2023%3A12%3A28.png)
  - ![image2025-5-19 23:12:46.png](/assets/01KJBZSRBWRW506KCEADZ31H61/image2025-5-19%2023%3A12%3A46.png)

尝试运行 num_generations=4, epoch=4

  - ![image2025-5-20 10:3:36.png](/assets/01KJBZSRBWRW506KCEADZ31H61/image2025-5-20%2010%3A3%3A36.png)
  - 没有明显的提升

尝试提高LR=1e-5:

  - ![image2025-5-21 10:46:53.png](/assets/01KJBZSRBWRW506KCEADZ31H61/image2025-5-21%2010%3A46%3A53.png)
  - 好像下限上升, 但没见明显提升
