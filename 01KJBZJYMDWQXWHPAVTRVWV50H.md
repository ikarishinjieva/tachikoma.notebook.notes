---
title: 20241114 - 使用ms-swift对Qwen2-VL进行微调 [4] - 修改loss
confluence_page_id: 3343121
created_at: 2024-11-14T11:56:16+00:00
updated_at: 2024-11-15T03:12:20+00:00
---

增加配置文件: 

```
{
    "完整的smiles表达式是:": [2.0, 2.0],
    "完整的smiles表达式是: ": [2.0, 2.0],
    "完整的SELFIES表达式是:": [2.0, 2.0],
    "完整的SELFIES表达式是: ": [2.0, 2.0]
}
``` 

运行命令: (base on v8-20241114-151036)

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
  --model_type qwen2-vl-7b-instruct \
  --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
  --sft_type lora \
  --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.json \
  --dataset_test_ratio 0.05 \
  --eval_steps 20 \
  --learning_rate 4e-4 \
  --num_train_epochs 5 \
  --logging_steps 20 \
  --use_loss_scale \
  --loss_scale_config_path /opt/huangyan/ms-swift/config/loss_scale_config.json \
" > train.log 2>&1
``` 

报错: 

```
[rank0]: Traceback (most recent call last):
[rank0]:   File "/opt/huangyan/ms-swift/swift/cli/sft.py", line 5, in <module>
[rank0]:     sft_main()
[rank0]:   File "/opt/huangyan/ms-swift/swift/utils/run_utils.py", line 32, in x_main
[rank0]:     result = llm_x(args, **kwargs)
[rank0]:   File "/opt/huangyan/ms-swift/swift/llm/sft.py", line 546, in llm_sft
[rank0]:     return trainer_train(args, model, template, train_dataset, val_dataset, callbacks=callbacks, msg=msg)
[rank0]:   File "/opt/huangyan/ms-swift/swift/llm/sft.py", line 496, in trainer_train
[rank0]:     trainer.train(training_args.resume_from_checkpoint)
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/mixin.py", line 493, in train
[rank0]:     res = super().train(resume_from_checkpoint, *args, **kwargs)
[rank0]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 2141, in train
[rank0]:     return inner_training_loop(
[rank0]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 2495, in _inner_training_loop
[rank0]:     tr_loss_step = self.training_step(model, inputs, num_items_in_batch)
[rank0]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 3613, in training_step
[rank0]:     loss = self.compute_loss(model, inputs, num_items_in_batch=num_items_in_batch)
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/trainers.py", line 167, in compute_loss
[rank0]:     outputs['loss'] = loss_func(outputs, **loss_kwargs)
[rank0]:   File "/opt/huangyan/ms-swift/swift/trainers/loss.py", line 75, in loss_scale_func
[rank0]:     shift_scale = shift_scale[masks]
[rank0]: IndexError: The shape of the mask [1, 491] at index 1 does not match the shape of the indexed tensor [1, 428] at index 1

``` 

代码分析后, 主要问题出现在这段代码中: 

![image2024-11-14 22:19:11.png](/assets/01KJBZJYMDWQXWHPAVTRVWV50H/image2024-11-14%2022%3A19%3A11.png)

给到模型的输入, 会将

但loss_scale计算得出的scale数组, 是按照没有增长的长度进行的, 所以计算出来scale数组长度 跟增长后的labels长度不匹配.

绕过: 由于仅在输入中有

![image2024-11-14 22:42:31.png](/assets/01KJBZJYMDWQXWHPAVTRVWV50H/image2024-11-14%2022%3A42%3A31.png)

训练效果: 

qwen2-vl-7b-instruct/v9-20241114-225100/runs

![image2024-11-15 11:11:26.png](/assets/01KJBZJYMDWQXWHPAVTRVWV50H/image2024-11-15%2011%3A11%3A26.png)

v9与v8相比, loss整体抬升, 意味着虽然增强了 smiles表达式部分的权重, 但训练路线并没有因为权重变化找到更好的路线
