---
title: 20241029 - 在新gpu服务器上, 进行模型训练 (化学分子式识别)
confluence_page_id: 3342857
created_at: 2024-10-29T06:45:29+00:00
updated_at: 2024-10-31T08:09:39+00:00
---

# 分布式训练的比较

最终结论: 几种分布式方式差异不大, 使用默认分布式

### 单节点

预估时间: 5:40:00

### 默认分布式

  - 预估时间: 2:42:30
  - 显存占用: 每显卡 18557MiB

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0,1 NCCL_P2P_DISABLE=1 NCCL_IB_DISABLE=1 HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 \
llamafactory-cli train \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-VL-7B-Instruct \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template qwen2_vl \
    --flash_attn auto \
    --dataset_dir data \
    --dataset decimer-img2smiles \
    --cutoff_len 1024 \
    --learning_rate 1e-05 \
    --num_train_epochs 10.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 4 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0.1 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 100 \
    --per_device_eval_batch_size 2" > train.log 2>&1 &
``` 

### accelerate (MULTI_GPU, 非fsdp)

  - 预估时间: 2:42:26
  - 显存占用: 每显卡 18579MiB

运行accelerate config生成配置文件: 

```
compute_environment: LOCAL_MACHINE
debug: true
distributed_type: MULTI_GPU
downcast_bf16: 'no'
enable_cpu_affinity: false
gpu_ids: 0,1
machine_rank: 0
main_training_function: main
mixed_precision: bf16
num_machines: 1
num_processes: 2
rdzv_backend: static
same_network: true
tpu_env: []
tpu_use_cluster: false
tpu_use_sudo: false
use_cpu: false
```
```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0,1 NCCL_P2P_DISABLE=1 NCCL_IB_DISABLE=1 HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 \
accelerate launch --config_file /root/.cache/huggingface/accelerate/default_config.yaml \
src/train.py \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-VL-7B-Instruct \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template qwen2_vl \
    --flash_attn auto \
    --dataset_dir data \
    --dataset decimer-img2smiles \
    --cutoff_len 1024 \
    --learning_rate 1e-05 \
    --num_train_epochs 10.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 4 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles \
    --bf16 True \
    --plot_loss True \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0.1 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 100 \
    --per_device_eval_batch_size 2" > train.log 2>&1 &
``` 

### deepspeed

  - 预估时间: 2:33:34
  - 显存占用: 每显卡 18313MiB

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0,1 NCCL_P2P_DISABLE=1 NCCL_IB_DISABLE=1 HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 \
deepspeed \
src/train.py \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-VL-7B-Instruct \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template qwen2_vl \
    --flash_attn auto \
    --dataset_dir data \
    --dataset decimer-img2smiles \
    --cutoff_len 1024 \
    --learning_rate 1e-05 \
    --num_train_epochs 10.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 4 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles_deepspeed \
    --bf16 True \
    --plot_loss True \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0.1 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 100 \
    --per_device_eval_batch_size 2 \
    --deepspeed /opt/huangyan/LLaMA-Factory/examples/deepspeed/ds_z0_config.json" > train.log 2>&1 &
``` 

# 训练decimer-img2smiles的全部数据 - 1

比起之前的实验, 调大了初始LR, 以及per_device_train_batch_size

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0,1 NCCL_P2P_DISABLE=1 NCCL_IB_DISABLE=1 HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 \
llamafactory-cli train \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-VL-7B-Instruct \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template qwen2_vl \
    --flash_attn auto \
    --dataset_dir data \
    --dataset decimer-img2smiles \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 10.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 8 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0.1 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 100 \
    --per_device_eval_batch_size 2" > train.log 2>&1 &
``` 

  - 每个GPU的显存使用: 19857 MB

效果: 

![image2024-10-29 19:42:50.png](/assets/01KJBZJ4ZJYX6CE5DC0AKEKGHH/image2024-10-29%2019%3A42%3A50.png)![image2024-10-29 19:42:55.png](/assets/01KJBZJ4ZJYX6CE5DC0AKEKGHH/image2024-10-29%2019%3A42%3A55.png) [附件: trainer_log.jsonl] 

实际测试: 

  - 输入: ![from-paper-5.png](/assets/01KJBZJ4ZJYX6CE5DC0AKEKGHH/from-paper-5.png)
  - 结果: C=CC(=O)OC1OC2CCOC(COC2=O)C1C (多次生成, 每次都不稳定)
  - 结果图片: 
    - ![image](https://cactus.nci.nih.gov/chemical/structure/C=CC(=O)OC1OC2CCOC(COC2=O)C1C/image?width=278&height=278)

下一步: 

  - 降低LR, 查看loss
  - 使用变形的图片训练, 查看效果
  - 让通用大模型生成标注, 标注+图片一同训练, 查看效果

# 训练decimer-img2smiles的全部数据 - 2

更换LR调度器, 使得训练后期的学习率更快调整

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0,1 NCCL_P2P_DISABLE=1 NCCL_IB_DISABLE=1 HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 \
llamafactory-cli train \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-VL-7B-Instruct \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template qwen2_vl \
    --flash_attn auto \
    --dataset_dir data \
    --dataset decimer-img2smiles \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 10.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 8 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type reduce_lr_on_plateau \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles.change_lr_scheduler \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0.1 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 100 \
    --per_device_eval_batch_size 2" > train.log 2>&1 &
``` 

报错: Received 1 death signal, shutting down workers. 怀疑是GPU内存超了.

继续执行命令, 断点续做

reduce_lr_on_plateau 没有其作用, 原因是其patient默认为10, 也就是10轮eval的loss结果不下降, 才会调整LR

需要将patient下调, 让LR下降更快. 

但必须使用yaml来设置 (命令行设置有bug), yaml文件: 

```
 ### model
model_name_or_path: Qwen/Qwen2-VL-7B-Instruct

### method
stage: sft
do_train: true
finetuning_type: lora
lora_target: all

### dataset
dataset: decimer-img2smiles
dataset_dir: data
template: qwen2_vl
cutoff_len: 1024
max_samples: 100000
preprocessing_num_workers: 16

### output
output_dir: saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles.change_lr_scheduler
logging_steps: 5
save_steps: 100
plot_loss: true

### train
per_device_train_batch_size: 8
gradient_accumulation_steps: 8
learning_rate: 5.0e-5
num_train_epochs: 10.0
lr_scheduler_type: reduce_lr_on_plateau
lr_scheduler_kwargs: '{"patience": 2}'
warmup_steps: 0
bf16: true
ddp_timeout: 180000000
include_num_input_tokens_seen: true
lora_rank: 8
lora_alpha: 16
lora_dropout: 0.1
optim: adamw_torch
max_grad_norm: 1.0
flash_attn: auto

### eval
val_size: 0.1
per_device_eval_batch_size: 2
eval_strategy: steps
eval_steps: 100

``` 

调低eval_steps  
  

命令: 

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0,1 NCCL_P2P_DISABLE=1 NCCL_IB_DISABLE=1 HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 \
llamafactory-cli train config/change_lr_scheduler.yaml" > train.log 2>&1 &
``` 

效果: 

![image2024-10-30 10:15:47.png](/assets/01KJBZJ4ZJYX6CE5DC0AKEKGHH/image2024-10-30%2010%3A15%3A47.png)![image2024-10-30 10:15:52.png](/assets/01KJBZJ4ZJYX6CE5DC0AKEKGHH/image2024-10-30%2010%3A15%3A52.png) [附件: trainer_log.jsonl] 

观察LR调整, 在1350 step进行了第一次调整, 每次调整一个数量级: 

```
{"current_steps": 1350, "total_steps": 1750, "loss": 0.1513, "learning_rate": 5e-05, "epoch": 7.67590618336887, "percentage": 77.14, "elapsed_time": "2:37:45", "remaining_time": "0:46:44", "throughput": 2539.32, "total_tokens": 24036672}
{"current_steps": 1350, "total_steps": 1750, "eval_loss": 0.3004102110862732, "epoch": 7.67590618336887, "percentage": 77.14, "elapsed_time": "2:38:57", "remaining_time": "0:47:06", "throughput": 2520.12, "total_tokens": 24036672}
{"current_steps": 1355, "total_steps": 1750, "loss": 0.1392, "learning_rate": 5e-06, "epoch": 7.704335465529495, "percentage": 77.43, "elapsed_time": "2:39:25", "remaining_time": "0:46:28", "throughput": 2522.1, "total_tokens": 24126144}
``` 

但调整前后, eval loss并不会继续下降, 也就是说loss不继续下降, 与LR调整不力无关

效果:

  - 输入: ![from-paper-5.png](/assets/01KJBZJ4ZJYX6CE5DC0AKEKGHH/from-paper-5.png)
  - 结果: C=CC(=O)OC1OC2OC(=O)C(C)(C)C2C1C (多次生成, 每次都不稳定)
  - 结果图片: 
    - ![image](https://cactus.nci.nih.gov/chemical/structure/C=CC(=O)OC1OC2OC(=O)C(C)(C)C2C1C/image?width=278&height=278)

# 训练decimer-img2smiles的全部数据 - 3

从1的基础上, 引入如下修改: 

  - warmup_steps: 500
  - lora_dropout: 0.2 
  - weight_decay: 0.01

配置文件: 

```
 ### model
model_name_or_path: Qwen/Qwen2-VL-7B-Instruct

### method
stage: sft
do_train: true
finetuning_type: lora
lora_target: all

### dataset
dataset: decimer-img2smiles
dataset_dir: data
template: qwen2_vl
cutoff_len: 1024
max_samples: 100000
preprocessing_num_workers: 16

### output
output_dir: saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles.change_weight_decay_lora_dropout
logging_steps: 5
save_steps: 100
plot_loss: true

### train
per_device_train_batch_size: 8
gradient_accumulation_steps: 8
learning_rate: 5.0e-5
num_train_epochs: 10.0
lr_scheduler_type: cosine
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
eval_steps: 100

### special
warmup_steps: 100
lora_dropout: 0.2
weight_decay: 0.01

``` 

相关知识: 

  - warmup预热, 是将LR从小逐步放大, 达到训练参数中的预设值, 预热阶段结束. 训练阶段再逐步减小LR进行收敛. 
    - 这样主要预防的问题是: 预热阶段主要改变的是学习率。在没有预热的情况下，训练一开始就使用初始学习率，由于模型的权重是随机初始化的，这可能会导致模型迅速朝着某个方向移动，进入一个局部最优解或导致梯度爆炸。
  - lora_dropout 的作用, 是避免过拟合
  - weight_decay 的作用:

````
2. **权重衰减惩罚了什么？ L2 正则化是什么意思？**

权重衰减惩罚的是**模型参数（权重）的大小**。  L2 正则化是实现权重衰减的一种常用方法。

* **L2 正则化:**  L2 正则化向损失函数添加一个正则化项，该项是所有模型参数的平方和乘以一个超参数（即权重衰减系数）。  这个正则化项鼓励模型学习更小的权重。

    ```
    Loss = Original Loss + (weight_decay / 2) * Σ(w_i^2)
    ```

    其中：
        * `Original Loss` 是原始的损失函数（例如交叉熵损失）。
        * `weight_decay` 是权重衰减系数，控制正则化的强度。
        * `w_i` 是模型的各个参数。

* **惩罚大权重:**  由于正则化项的存在，模型在优化过程中不仅要最小化原始损失，还要最小化权重的平方和。  这会使得模型倾向于学习更小、更分散的权重。

* **为什么惩罚大权重可以防止过拟合？**  过拟合通常发生在模型过于复杂，能够完美地拟合训练数据，包括噪声的情况下。  大的权重通常意味着模型对某些特征过于敏感，容易受到噪声的影响。  通过限制权重的大小，L2 正则化可以降低模型的复杂度，使其对噪声 less sensitive，从而提高模型的泛化能力，防止过拟合。

总而言之，权重衰减通过 L2 正则化来惩罚模型参数的大小，鼓励模型学习更小、更分散的权重，从而降低模型复杂度，提高泛化能力，防止过拟合。
````

    - weight_decay 是否起作用: 其与origin loss应当在同样的数量级, 否则会影响过大或者过小. 但目前训练工具没有现成的方法 (除了改代码) 进行信息确认. 

训练效果: 

![image2024-10-30 14:38:10.png](/assets/01KJBZJ4ZJYX6CE5DC0AKEKGHH/image2024-10-30%2014%3A38%3A10.png)![image2024-10-30 14:38:15.png](/assets/01KJBZJ4ZJYX6CE5DC0AKEKGHH/image2024-10-30%2014%3A38%3A15.png)

曲线跟之前没有差别, 这几个参数解决过拟合问题的效果不佳

TODO: 可以尝试降低初始LR

# 训练decimer-img2smiles的全部数据 - 4 - 数据增强

使用Img2Mol中的图片变换方法: [Untitled-1.py](/assets/01KJBZJ4ZJYX6CE5DC0AKEKGHH/Untitled-1.py)

对训练数据进行3倍增值变换

配置文件:

```
### model
model_name_or_path: Qwen/Qwen2-VL-7B-Instruct

### method
stage: sft
do_train: true
finetuning_type: lora
lora_target: all

### dataset
dataset: decimer-img2smiles,decimer-img2smiles-enhanced
dataset_dir: data
template: qwen2_vl
cutoff_len: 1024
preprocessing_num_workers: 16

### output
output_dir: saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles.add_enhanced_data
logging_steps: 5
save_steps: 100
plot_loss: true

### train
per_device_train_batch_size: 8
gradient_accumulation_steps: 8
learning_rate: 5.0e-5
num_train_epochs: 10.0
lr_scheduler_type: cosine
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
eval_steps: 100

### special
max_samples: 1000000
warmup_steps: 100
lora_dropout: 0.2
weight_decay: 0.01
report_to: tensorboard
logging_dir: tensorboard_logging/
logging_steps: 100

``` 

增加了tensorboard输出: http://127.0.0.1:6006

结果: 

![image2024-10-31 15:18:13.png](/assets/01KJBZJ4ZJYX6CE5DC0AKEKGHH/image2024-10-31%2015%3A18%3A13.png)

eval loss最终略小于0.2, 但没有进一步提升. 比起没有增强数据的时候, 没有提升.

从实际测试效果, 也与 没有增强数据的时候类似

后续: 

  1. 图像增强的方式是否有问题, 导致增强后的效果没有提升? 
  2. 将图片的描述文字加入训练, 查看效果
