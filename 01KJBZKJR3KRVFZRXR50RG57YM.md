---
title: 20241115 - 使用ms-swift对Qwen2-VL进行微调 [5] - 解决过拟合状态
confluence_page_id: 3343143
created_at: 2024-11-15T08:34:38+00:00
updated_at: 2024-11-19T11:44:49+00:00
---

# 之前的结论

根据之前的尝试, 得出以下结论: 

  1. [20241114 - 使用ms-swift对Qwen2-VL进行微调 [3] - 使用更多数据], 使用更多数据, 对效果没有改善
  2. [20241114 - 使用ms-swift对Qwen2-VL进行微调 [4] - 修改loss], 增强结论的loss权重, 对效果没有改善
  3. [20241114 - 使用ms-swift对Qwen2-VL进行微调 [2] - 使用SELFIES替代SMILES], 还在等待结果

# 建立基线

缩减到1000数据, 建立基线: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
  --model_type qwen2-vl-7b-instruct \
  --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
  --sft_type lora \
  --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.json \
  --dataset_test_ratio 0.05 \
  --eval_steps 20 \
  --learning_rate 4e-4 \
  --num_train_epochs 5 \
  --logging_steps 20" > train.log 2>&1
``` 

qwen2-vl-7b-instruct/v12-20241115-164941/runs:

![image2024-11-15 18:26:5.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-15%2018%3A26%3A5.png)

已经出现了过拟合

# 增强数据

用增强脚本进行数据生成 (/opt/huangyan/LLaMA-Factory/data_gen/enhance_decimer_images.ipynb), 每个数据增加三个增强变换

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
  --model_type qwen2-vl-7b-instruct \
  --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
  --sft_type lora \
  --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.enhanced.json \
  --dataset_test_ratio 0.05 \
  --eval_steps 20 \
  --learning_rate 4e-4 \
  --num_train_epochs 5 \
  --logging_steps 20" > train.log 2>&1
``` 

qwen2-vl-7b-instruct/v13-20241115-183048/runs:

![image2024-11-15 19:52:0.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-15%2019%3A52%3A0.png)

效果异常好. loss完全下降

作图: grad_norm * LR , 和 (eval-train)/eval

![image2024-11-15 19:52:58.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-15%2019%3A52%3A58.png)

过拟合出现在 step 400-500, (整图长度为 step 1000), 过拟合出现后, (grad_norm * LR)开始下降, 模型开始进行精细调整

注意: 怀疑是数据增值以后, eval中的数据是train中数据的变形, 而不是完全独立的数据, 这个实验需要重新进行.

调整生成脚本, 将数据的5%作为校验集 (不增殖), 训练数据则需要增殖. 运行命令: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
  --model_type qwen2-vl-7b-instruct \
  --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
  --sft_type lora \
  --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.enhanced.train.json \
  --val_dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.enhanced.val.json \
  --eval_steps 20 \
  --learning_rate 4e-4 \
  --num_train_epochs 5 \
  --logging_steps 20" > train.log 2>&1
``` 

  

效果: 

qwen2-vl-7b-instruct/v28-20241117-005348/runs

![image2024-11-17 11:28:56.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-17%2011%3A28%3A56.png)

之前的怀疑没有错. 在本次实验中, 数据增殖前后, 过拟合的时间点和程度相当.

也就是说: 简单的图片增殖并不能改善训练效果, Qwen2-VL中已经包括了简单图片变换的处理

# 使用SELFIES表达式替换Smiles表达式

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
  --model_type qwen2-vl-7b-instruct \
  --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
  --sft_type lora \
  --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.SELFIES.data-1000.json \
  --dataset_test_ratio 0.05 \
  --eval_steps 20 \
  --learning_rate 4e-4 \
  --num_train_epochs 5 \
  --logging_steps 20" > train.log 2>&1
``` 

训练出现报错: 

```
[WARNING:swift] Current length of row(2162) is larger than the max_length(2048), deleted.
[WARNING:swift] Current length of row(3006) is larger than the max_length(2048), deleted.
[WARNING:swift] Current length of row(2113) is larger than the max_length(2048), deleted.
``` 

效果: qwen2-vl-7b-instruct/v14-20241115-195537/runs

![image2024-11-16 13:43:40.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-16%2013%3A43%3A40.png)

对于SELFIES数据的训练, loss会低于SMILES数据的训练. 其中可能存在的问题是整个COT过程的长度增大 (更啰嗦), 导致loss的计算中分母在变大.

考虑更新eval函数, 使其只计算最终表达式的相似程度, 来体现真实的状况.

# 使用不同的LR进行训练

v15-20241115-202651: LR=4e-5

v16-20241115-204750: LR=8e-5

v17-20241115-210855: LR=2e-4

![image2024-11-16 13:54:33.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-16%2013%3A54%3A33.png)

v15/v16 都不会出现过拟合, 在验证集上的表现比基线更好

v17开始出现了过拟合

(基线) V12: LR=4-e4

v18-20241115-212958: LR=8e-4

v19-20241115-215055: LR=2e-3

![image2024-11-16 13:57:15.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-16%2013%3A57%3A15.png)

从v18开始, 过早出现过拟合, 整个eval loss曲线都会比基线高

v20-20241115-221153: LR=4e-3

v21-20241115-223247: LR=8e-3

v22-20241115-225349: LR=2e-2

![image2024-11-16 13:59:37.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-16%2013%3A59%3A37.png)

从v20开始, loss起始过高, 梯度爆炸

# 拉长epoch

将LR=8e-5 (v18, 目前过拟合最晚的配置), epoch拉长到20: 

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
>   --model_type qwen2-vl-7b-instruct \
>   --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
>   --sft_type lora \
>   --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.enhanced.train.json \
>   --val_dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.enhanced.val.json \
>   --eval_steps 20 \
>   --learning_rate 8e-5 \
>   --num_train_epochs 20 \
>   --logging_steps 20" > train.log 2>&1
``` 

  
qwen2-vl-7b-instruct/v29-20241117-114737/runs:

![image2024-11-17 20:46:38.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-17%2020%3A46%3A38.png)

过拟合节点不变, 后面过拟合情况会越来越严重

# 使用不同的优化器

v12 (基线): optim=adamw_torch

v23-20241116-142657: optim=sgd

v24-20241116-144749: optim=adafactor

![image2024-11-16 18:10:31.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-16%2018%3A10%3A31.png)

adafactor和adamw_torch的曲线差不多.

sgd完全没有作用, 可能需要设置动量momentum :question:

v25-20241116-182949: optim=rmsprop

![image2024-11-17 0:45:35.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-17%200%3A45%3A35.png)

rmsprop的loss下降更缓慢, grad更大 (学得更多, 但是没用)

# 测试predict_with_generate - (建立更合适的eval指标)

```
bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
  --model_type qwen2-vl-7b-instruct \
  --model_id_or_path qwen/Qwen2-VL-7B-Instruct \
  --sft_type lora \
  --dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.enhanced.train.json \
  --val_dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.enhanced.val.json \
  --eval_steps 20 \
  --learning_rate 8e-5 \
  --num_train_epochs 20 \
  --logging_steps 20 \
  --predict_with_generate \
  " > train.log 2>&1
``` 

报错: 

```
[rank1]: Traceback (most recent call last):
[rank1]:   File "/opt/huangyan/ms-swift/swift/cli/sft.py", line 5, in <module>
[rank1]:     sft_main()
[rank1]:   File "/opt/huangyan/ms-swift/swift/utils/run_utils.py", line 32, in x_main
[rank1]:     result = llm_x(args, **kwargs)
[rank1]:   File "/opt/huangyan/ms-swift/swift/llm/sft.py", line 546, in llm_sft
[rank1]:     return trainer_train(args, model, template, train_dataset, val_dataset, callbacks=callbacks, msg=msg)
[rank1]:   File "/opt/huangyan/ms-swift/swift/llm/sft.py", line 496, in trainer_train
[rank1]:     trainer.train(training_args.resume_from_checkpoint)
[rank1]:   File "/opt/huangyan/ms-swift/swift/trainers/mixin.py", line 493, in train
[rank1]:     res = super().train(resume_from_checkpoint, *args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 2141, in train
[rank1]:     return inner_training_loop(
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 2562, in _inner_training_loop
[rank1]:     self._maybe_log_save_evaluate(tr_loss, grad_norm, model, trial, epoch, ignore_keys_for_eval)
[rank1]:   File "/opt/huangyan/ms-swift/swift/trainers/mixin.py", line 569, in _maybe_log_save_evaluate
[rank1]:     super()._maybe_log_save_evaluate(tr_loss, *args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 3018, in _maybe_log_save_evaluate
[rank1]:     metrics = self._evaluate(trial, ignore_keys_for_eval)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 2972, in _evaluate
[rank1]:     metrics = self.evaluate(ignore_keys=ignore_keys_for_eval)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer_seq2seq.py", line 195, in evaluate
[rank1]:     return super().evaluate(eval_dataset, ignore_keys=ignore_keys, metric_key_prefix=metric_key_prefix)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 4009, in evaluate
[rank1]:     output = eval_loop(
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 4203, in evaluation_loop
[rank1]:     losses, logits, labels = self.prediction_step(model, inputs, prediction_loss_only, ignore_keys=ignore_keys)
[rank1]:   File "/opt/huangyan/ms-swift/swift/trainers/trainers.py", line 100, in prediction_step
[rank1]:     generated_tokens = self.model.generate(**generate_inputs, **gen_kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/peft/peft_model.py", line 1638, in generate
[rank1]:     outputs = self.base_model.generate(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/torch/utils/_contextlib.py", line 116, in decorate_context
[rank1]:     return func(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/generation/utils.py", line 1988, in generate
[rank1]:     self._validate_model_kwargs(model_kwargs.copy())
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/generation/utils.py", line 1371, in _validate_model_kwargs
[rank1]:     raise ValueError(
[rank1]: ValueError: The following `model_kwargs` are not used by the model: ['_data'] (note: typos in the generate arguments will also show up in this list)
``` 

在Seq2SeqTrainer.prediction_step中, 临时去掉_data: 

![image2024-11-18 0:59:14.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-18%200%3A59%3A14.png)

会报错: 

```
[rank1]: Traceback (most recent call last):
[rank1]:   File "/opt/huangyan/ms-swift/swift/cli/sft.py", line 5, in <module>
[rank1]:     sft_main()
[rank1]:   File "/opt/huangyan/ms-swift/swift/utils/run_utils.py", line 32, in x_main
[rank1]:     result = llm_x(args, **kwargs)
[rank1]:   File "/opt/huangyan/ms-swift/swift/llm/sft.py", line 546, in llm_sft
[rank1]:     return trainer_train(args, model, template, train_dataset, val_dataset, callbacks=callbacks, msg=msg)
[rank1]:   File "/opt/huangyan/ms-swift/swift/llm/sft.py", line 496, in trainer_train
[rank1]:     trainer.train(training_args.resume_from_checkpoint)
[rank1]:   File "/opt/huangyan/ms-swift/swift/trainers/mixin.py", line 493, in train
[rank1]:     res = super().train(resume_from_checkpoint, *args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 2141, in train
[rank1]:     return inner_training_loop(
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 2562, in _inner_training_loop
[rank1]:     self._maybe_log_save_evaluate(tr_loss, grad_norm, model, trial, epoch, ignore_keys_for_eval)
[rank1]:   File "/opt/huangyan/ms-swift/swift/trainers/mixin.py", line 569, in _maybe_log_save_evaluate
[rank1]:     super()._maybe_log_save_evaluate(tr_loss, *args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 3018, in _maybe_log_save_evaluate
[rank1]:     metrics = self._evaluate(trial, ignore_keys_for_eval)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 2972, in _evaluate
[rank1]:     metrics = self.evaluate(ignore_keys=ignore_keys_for_eval)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer_seq2seq.py", line 195, in evaluate
[rank1]:     return super().evaluate(eval_dataset, ignore_keys=ignore_keys, metric_key_prefix=metric_key_prefix)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 4009, in evaluate
[rank1]:     output = eval_loop(
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/trainer.py", line 4203, in evaluation_loop
[rank1]:     losses, logits, labels = self.prediction_step(model, inputs, prediction_loss_only, ignore_keys=ignore_keys)
[rank1]:   File "/opt/huangyan/ms-swift/swift/trainers/trainers.py", line 101, in prediction_step
[rank1]:     generated_tokens = self.model.generate(**generate_inputs, **gen_kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/peft/peft_model.py", line 1638, in generate
[rank1]:     outputs = self.base_model.generate(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/torch/utils/_contextlib.py", line 116, in decorate_context
[rank1]:     return func(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/generation/utils.py", line 2231, in generate
[rank1]:     result = self._sample(
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/generation/utils.py", line 3222, in _sample
[rank1]:     outputs = self(**model_inputs, return_dict=True)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1736, in _wrapped_call_impl
[rank1]:     return self._call_impl(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1747, in _call_impl
[rank1]:     return forward_call(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/models/qwen2_vl/modeling_qwen2_vl.py", line 1726, in forward
[rank1]:     outputs = self.model(
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1736, in _wrapped_call_impl
[rank1]:     return self._call_impl(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1747, in _call_impl
[rank1]:     return forward_call(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/models/qwen2_vl/modeling_qwen2_vl.py", line 1159, in forward
[rank1]:     layer_outputs = decoder_layer(
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1736, in _wrapped_call_impl
[rank1]:     return self._call_impl(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1747, in _call_impl
[rank1]:     return forward_call(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/models/qwen2_vl/modeling_qwen2_vl.py", line 906, in forward
[rank1]:     hidden_states, self_attn_weights, present_key_value = self.self_attn(
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1736, in _wrapped_call_impl
[rank1]:     return self._call_impl(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/torch/nn/modules/module.py", line 1747, in _call_impl
[rank1]:     return forward_call(*args, **kwargs)
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/models/qwen2_vl/modeling_qwen2_vl.py", line 800, in forward
[rank1]:     query_states, key_states = apply_multimodal_rotary_pos_emb(
[rank1]:   File "/usr/local/lib/python3.10/dist-packages/transformers/models/qwen2_vl/modeling_qwen2_vl.py", line 243, in apply_multimodal_rotary_pos_emb
[rank1]:     q_embed = (q * cos) + (rotate_half(q) * sin)
[rank1]: RuntimeError: The size of tensor a (170) must match the size of tensor b (436) at non-singleton dimension 2
``` 

手工模拟Seq2SeqTrainer.prediction_step, 直接使用Qwen2VL模型, 然后比较两者区别: 

  - [qwen2vl.ipynb](/assets/01KJBZKJR3KRVFZRXR50RG57YM/qwen2vl.ipynb)
  - 主要比较输入的区别: 发现多出_data和position_ids, 将其从输入中去掉

![image2024-11-18 23:54:3.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-18%2023%3A54%3A3.png)

运行结果: 

qwen2-vl-7b-instruct/v26-20241119-000435/runs

![image2024-11-19 10:50:11.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-19%2010%3A50%3A11.png)

结论: 

  - predict_with_generate非常慢, 9小时训练13%, 主要消耗在eval

```
Train:  13%|█▎        | 620/4760 [9:37:52<4:00:42,  3.49s/it]
Val: 100%|██████████| 23/23 [11:27<00:00, 29.87s/it]
```

  - predict_with_generate影响的是 compute_metrics, 其不影响训练过程, 独立于loss
  - 从曲线图上看, eval/bleu-4 在过拟合点后, 没有明显的增长趋势, 应征了模型确实过拟合的结论 (train学得越来越多, 但eval并没有学习到)

# 使用SELFIES表达式, 增加手工评估脚本, 来评估其是否有用

跑smiles和SELFIES两个训练: 

```
#!/bin/bash

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.val.json \
--eval_steps 20 \
--learning_rate 8e-5 \
--num_train_epochs 10 \
--logging_steps 20" > train_1.log 2>&1

sleep 300

bash -c "HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 CUDA_VISIBLE_DEVICES=0,1 NPROC_PER_NODE=2 swift sft \
--model_type qwen2-vl-7b-instruct \
--model_id_or_path qwen/Qwen2-VL-7B-Instruct \
--sft_type lora \
--dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.SELFIES.train.json \
--val_dataset /opt/huangyan/LLaMA-Factory/data/decimer-img2smiles.cot.ms-swift.data-1000.SELFIES.val.json \
--eval_steps 20 \
--learning_rate 8e-5 \
--num_train_epochs 10 \
--logging_steps 20" > train_2.log 2>&1

``` 

v30-20241119-112757: smiles, 1000数据, 10 epoch, LR=8e-5

v31-20241119-120543: SELFIES, 1000数据, 10 epoch, LR=8e-5

![image2024-11-19 13:9:8.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-19%2013%3A9%3A8.png)

使用手工评估脚本: [evaluate_smiles_selfies.1.ipynb](/assets/01KJBZKJR3KRVFZRXR50RG57YM/evaluate_smiles_selfies.1.ipynb)

v30 (smiles) 的结果: 

![image2024-11-19 19:12:41.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-19%2019%3A12%3A41.png)

v31 (selfies) 的结果: 

![image2024-11-19 19:12:53.png](/assets/01KJBZKJR3KRVFZRXR50RG57YM/image2024-11-19%2019%3A12%3A53.png)

selfies 的健壮性更好, 但结构相似性更低
