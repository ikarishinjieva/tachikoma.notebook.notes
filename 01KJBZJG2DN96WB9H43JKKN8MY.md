---
title: 20241107 - 训练模型(化学分子式识别) - 尝试使用CoT
confluence_page_id: 3342987
created_at: 2024-11-06T16:12:17+00:00
updated_at: 2024-11-07T15:03:52+00:00
---

# 使用Claude生成CoT过程

````
我有一张图片如附件, 其对应的smiles表达式如下: 
```
CCOC(=O)c1c(NC2=NS(=O)(=O)c3ccccc32)sc2c1CCC(C)C2
```

你需要输出你如何分析这张图片, 如何一步一步构建smiles表达式, 比如图片的哪一部分是什么, 对应的smiles是什么, 为什么, 如何拼接到最终的smiles表达式中.
最终告诉我完整的smiles表达式是什么.

输出格式要求: 
- 如何分析图片, 构建smiles表达式
- 完整的smiles表达式是: ......
```` 

答案: 

```
让我帮你分析这个分子结构和构建SMILES表达式:

如何分析图片和构建SMILES表达式:

1. 首先识别主要的环系统:
   - 左侧有一个苯并异噻唑-1,1-二氧化物环系统 (benzisothiazole 1,1-dioxide)
   - 右侧有一个2-氨基噻吩环 (2-aminothiophene)和六元环 (cyclohexane)的稠合系统

2. 从左边开始构建:
   - 苯并异噻唑环系统: c3ccccc32 (苯环部分)
   - 加入SO2基团和氮原子: NS(=O)(=O)c3ccccc32

3. 向右构建:
   - 连接NH桥: NC2=NS(=O)(=O)c3ccccc32
   - 噻吩环连接: sc2c1 (其中1是为了后续连接)

4. 右侧结构:
   - 六元环带甲基: CCC(C)C2
   - 酯基: CCOC(=O)

5. 将所有部分连接起来，注意:
   - 酯基连接到噻吩环上
   - 六元环与噻吩环稠合
   - 保持正确的连接点编号

完整的SMILES表达式是:
CCOC(=O)c1c(NC2=NS(=O)(=O)c3ccccc32)sc2c1CCC(C)C2

这个SMILES表达式准确地描述了图片中显示的分子结构，包括所有环系统、取代基和连接方式。每个原子的连接性和化学环境都得到了正确表示。
``` 

# 2000数据训练

配置: 

```
### model
model_name_or_path: Qwen/Qwen2-VL-7B-Instruct

### method
stage: sft
do_train: true
finetuning_type: lora
lora_target: all

### dataset
dataset: decimer-img2smiles_cot
dataset_dir: data
template: qwen2_vl
cutoff_len: 1024
preprocessing_num_workers: 16

### output
output_dir: saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles_cot_2000
save_steps: 100
plot_loss: true

### train
per_device_train_batch_size: 2
gradient_accumulation_steps: 8
bf16: true
ddp_timeout: 180000000
include_num_input_tokens_seen: true
lora_rank: 8
lora_alpha: 16
optim: adamw_torch
max_grad_norm: 1.0
flash_attn: auto

### eval
val_size: 0.1
per_device_eval_batch_size: 2
eval_strategy: steps

### special
max_samples: 2000
warmup_steps: 100
lora_dropout: 0.2
weight_decay: 0.01
report_to: tensorboard
logging_dir: tensorboard_logging/img2smiles_cot_2000
logging_steps: 20
eval_steps: 20
num_train_epochs: 30.0
learning_rate: 4.0e-4
lr_scheduler_type: cosine

 
``` 

![image2024-11-7 21:55:49.png](/assets/01KJBZJG2DN96WB9H43JKKN8MY/image2024-11-7%2021%3A55%3A49.png)

出现了过拟合

# 3000数据训练

![image2024-11-7 22:57:46.png](/assets/01KJBZJG2DN96WB9H43JKKN8MY/image2024-11-7%2022%3A57%3A46.png)

3000比2000的过拟合现象缓和, 但转折点都是warmup结束.

需要尝试: 

  1. 调低LR
  2. 延长warmup, 观察拐点
