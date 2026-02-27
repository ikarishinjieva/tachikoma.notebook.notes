---
title: 20250922 - SQL优化的GRPO训练 - 生成具有挑战的case
confluence_page_id: 4358604
created_at: 2025-09-21T23:24:46+00:00
updated_at: 2025-09-27T04:42:07+00:00
---

# 前继

在之前的探索中 ([20250821 - SQL优化的GRPO训练 - 重新制作数据生成过程]), 发现需要生成对模型具有挑战的case, 才能平衡 SQL的各种劣化方向的 处理方法的选择. 

具有挑战的case, 需要满足如下要求: 

  1. 提示词足够友好, 给模型提供思考步骤模板, 让基模型能自己思考
  2. 让基模型自己能思考, 有小概率找到正确解, 大概率是错误解
  3. 对于多种思考方向, 生成综合性的复杂case (判断基模型对于简单的单一的case具备基础能力)

# 生成脚本

commit: 9660fd8f7d72a58e5fcadec9bbaeb7d2bca761f6

使用脚本: make_train_dataset.v4/

发现对于Qwen3-14b, 出现两个问题: 

  - 对于CTE的子查询, 并不能列表在动作一的检查表的列表中
  - 以当前的提示词, 基模型不会使用"级联删除"的逻辑

修改提示词: 

![image2025-9-22 16:34:10.png](/assets/01KJC00DJ026W6BGSW0TEW00GJ/image2025-9-22%2016%3A34%3A10.png)

  1. 改成"可选的动作逻辑"
  2. 将级联删除这一项放在最前面

可以让基模型使用"级联删除"的逻辑

生成数据以后, 评估基模型对这些数据的推理能力, 没有一个case能达到1.0. 每一个case的reward区间都跨了0.2-0.4左右. 

检查case为什么不能达到1.0, 有以下几个原因: 

  - 在动作1中, 并不能穷举所有的表, 会遗漏一些表
  - 基模型使用了如下输出, reward函数解析错误: 
    - 但其提供了一个对"级联删除"的良好的描述

```
- 动作3: 检查语句(g_mid子查询-2.1.1)的输出字段(creation_date): 被 g_details子查询-2.1 作为(选择字段)引用: (SELECT g_mid.creation_date...), 该字段的输出在父语句中被引用, 而父语句的对应字段可以删除, 因此该字段也可以删除
``` 
    - 旧语句: 

```
- "检查语句({{语句名}})的输出字段({{字段内容}}): 被 同级其他语句或父级各级语句({{语句名列表}})作为({{选择字段/表连接列/计算列/条件列/等}})引用: ({{被引用的片段}}), 被引用处已经被标记成可删除, 当前输出字段已没有其他引用, 可以删除"
```

重新生成数据, 使用25条数据, 进行GRPO:

  - 生成数据的方式:   

    - 使用8个挑战, 让大模型生成SQL 和 优化后的SQL, 并生成中间步骤
    - 让基模型对训练数据的prompt进行reward评分 (目前是人工审核, 并没有进行自动筛选)
  - 第一次使用GAS=2, (灰色曲线), 导致OOM. 曲线在eval上上升趋势良好
  - 第二次使用GAS=1, (橘色曲线), 未跑完
    - ![image2025-9-23 18:39:14.png](/assets/01KJC00DJ026W6BGSW0TEW00GJ/image2025-9-23%2018%3A39%3A14.png)
    - 分析completion: 
      - 最高分到0.94, 且频繁出现
      - v1.months 有概率正确处理, 级联删除的分析 大概率正确
      - 不能正确处理的是: 在一个select中, 同时选择了计算列和成分列, 成分列不能正确删除
    - epoch设置的很长 (epoch=20), 当前这一段有效曲线 (epoch=7)的LR, 从4e-5到3.3e-5

重新生成数据: 

  - 调整生成数据的方式:
    - commit: 4cdfe5bf314f5ed2107b2e18c918a80944b23dad
    - 随机选取某几个挑战, 每个挑战都有自己的概率
    - 将 "在一个select中, 同时选择了计算列和成分列, 成分列不能正确删除" 增加为一个单独的挑战
    - 总40个数据
  - 尝试GAS=2, 缩短max_tokens, 避开OOM
    - 上一次训练日志中, 训练数据的token统计: "[INFO:swift] Dataset Token Length: 6510.400000±523.402713, min=5202.000000, max=7656.000000, size=25"
    - 尝试参数: 

```
--max_length 8192 \
--max_completion_length 8192 \
--vllm_max_model_len 8192 \
--vllm_gpu_memory_utilization 0.85 \
--num_generations 4
```

    - (红色曲线为 epoch=4, 蓝色曲线为epoch=8)

    - ![image2025-9-24 10:43:23.png](/assets/01KJC00DJ026W6BGSW0TEW00GJ/image2025-9-24%2010%3A43%3A23.png)

      - 达到eval/reward极值的时间差不多, 往后开始下降
      - 除了这两次训练, 其他训练均碰到OOM, 这一点很奇怪

只选取token < 6000的训练数据

  - 总34个数据
  - 两阶段训练 (GAS=4 , (OOM中断, 续训), GAS=1): 
    - ![image2025-9-25 10:50:5.png](/assets/01KJC00DJ026W6BGSW0TEW00GJ/image2025-9-25%2010%3A50%3A5.png)
    - 训练效果一般, eval/reward 只到0.8
  - 一阶段训练 (GAS=1)
    - ![image2025-9-25 10:52:32.png](/assets/01KJC00DJ026W6BGSW0TEW00GJ/image2025-9-25%2010%3A52%3A32.png)
    - 收敛很快, 但后续不足
  - 一阶段训练, lora_drop=0.15, 蓝色曲线
    - ![image2025-9-25 13:9:34.png](/assets/01KJC00DJ026W6BGSW0TEW00GJ/image2025-9-25%2013%3A9%3A34.png)
    - 未见明显差异
  - 一阶段训练, 去掉CHORD, 
    - ![image2025-9-26 23:40:12.png](/assets/01KJC00DJ026W6BGSW0TEW00GJ/image2025-9-26%2023%3A40%3A12.png)
    - 极值出现的很晚, 但极值最高值没有改善 (评估数据中, 仍然会不使用一些优化手段, 比如判断条件列等)

想法:

  - 本次数据给各种挑战按照概率进行组合, 上次数据(25条数据)是将所有挑战全部组合. 
    - 上次数据: /data/huangyan/sql-ai-2/make_train_dataset.v4/train.0923.25.jsonl
    - 本次数据: /data/huangyan/sql-ai-2/make_train_dataset.v4/train.0925.34.jsonl
  - 将 上次的数据 放在本次训练的模型中评估一下, 如果reward仍有提升空间, 考虑将两次数据合并在一起训练
  - 总体: 找到难度更高的数据 (如何定义难度)

在续训中(绿色曲线), 使用第一阶段(红色曲线, 续训取这一阶段的高点)没有满分的case: 

  - ![image2025-9-25 21:51:35.png](/assets/01KJC00DJ026W6BGSW0TEW00GJ/image2025-9-25%2021%3A51%3A35.png)
  - 未见明显差异

使用线性的LR, 从8e-5开始, eval明显变差: 

![image2025-9-26 10:25:40.png](/assets/01KJC00DJ026W6BGSW0TEW00GJ/image2025-9-26%2010%3A25%3A40.png)

# 一直OOM

报错: 

```
INFO 09-27 11:37:10 [executor_base.py:204] It took 0.677008 seconds to wake up tags ['weights'].
[rank0]: Traceback (most recent call last):
[rank0]:   File "/data/huangyan/ms-swift/swift/cli/rlhf.py", line 5, in <module>
[rank0]:     rlhf_main()
[rank0]:   File "/data/huangyan/ms-swift/swift/llm/train/rlhf.py", line 217, in rlhf_main
[rank0]:     return SwiftRLHF(args).main()
[rank0]:   File "/data/huangyan/ms-swift/swift/llm/base.py", line 49, in main
[rank0]:     result = self.run()
[rank0]:   File "/data/huangyan/ms-swift/swift/llm/train/sft.py", line 195, in run
[rank0]:     return self.train(trainer)
[rank0]:   File "/data/huangyan/ms-swift/swift/llm/train/sft.py", line 243, in train
[rank0]:     trainer.train(trainer.args.resume_from_checkpoint)
[rank0]:   File "/data/huangyan/ms-swift/swift/trainers/mixin.py", line 674, in train
[rank0]:     res = super().train(*args, **kwargs)
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 2328, in train
[rank0]:     return inner_training_loop(
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 2672, in _inner_training_loop
[rank0]:     tr_loss_step = self.training_step(model, inputs, num_items_in_batch)
[rank0]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 1969, in training_step
[rank0]:     return super().training_step(model, inputs, num_items_in_batch)
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 4003, in training_step
[rank0]:     inputs = self._prepare_inputs(inputs)
[rank0]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/utils.py", line 169, in wrapper
[rank0]:     return func(self, *args, **kwargs)
[rank0]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 433, in _prepare_inputs
[rank0]:     generation_batch = self._generate_and_score_completions(generation_batch)
[rank0]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/utils.py", line 169, in wrapper
[rank0]:     return func(self, *args, **kwargs)
[rank0]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 840, in _generate_and_score_completions
[rank0]:     inputs = self._generate_completions(inputs)
[rank0]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 821, in _generate_completions
[rank0]:     results = self._fast_infer(inputs)
[rank0]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 767, in _fast_infer
[rank0]:     self.engine.engine.wake_up(**kwargs)
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/v1/engine/llm_engine.py", line 291, in wake_up
[rank0]:     self.engine_core.wake_up(tags)
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/v1/engine/core_client.py", line 278, in wake_up
[rank0]:     self.engine_core.wake_up(tags)
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/v1/engine/core.py", line 388, in wake_up
[rank0]:     self.model_executor.wake_up(tags)
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/executor/executor_base.py", line 202, in wake_up
[rank0]:     self.collective_rpc("wake_up", kwargs=dict(tags=tags))
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/executor/uniproc_executor.py", line 58, in collective_rpc
[rank0]:     answer = run_method(self.driver_worker, method, args, kwargs)
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/utils/__init__.py", line 3060, in run_method
[rank0]:     return func(*args, **kwargs)
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/v1/worker/gpu_worker.py", line 125, in wake_up
[rank0]:     allocator.wake_up(tags)
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/device_allocator/cumem.py", line 230, in wake_up
[rank0]:     create_and_map(handle)
[rank0]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/device_allocator/cumem.py", line 78, in create_and_map
[rank0]:     python_create_and_map(*allocation_handle)
[rank0]: RuntimeError: CUDA Error: out of memory at /workspace/csrc/cumem_allocator.cpp:62

``` 

通过调整vllm_max_num_seqs=1可解决
