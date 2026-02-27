---
title: 20241031 - 训练模型(化学分子式识别) - 增加图片的文字描述
confluence_page_id: 3342904
created_at: 2024-10-31T08:15:06+00:00
updated_at: 2024-11-06T16:28:24+00:00
---

使用模型 Claude-3.5-sonnet

描述提示词: 

```
图中是一个化学分子结构, 我希望你详细描述这个化学分子结构 的 结构/构成/特征/位置信息/等 你从图片上识别出来的信息. 你描述这些信息的时候, 需要带上位置信息. 这些信息要帮助一个OCR系统来识别这个分子结构.
用中文回答.
``` 

样例描述: 

```
这个化学分子结构图显示了一个复杂的有机化合物。我将详细描述其结构、构成和特征，并附带位置信息：

1. 中心是一个六元环结构，可能是一个吡啶或嘧啶环。

2. 在六元环的顶部（12点位置），有一个 -NH2 基团。

3. 顺时针方向，在大约1点钟位置，有一个 -NO2 基团，其中氧原子朝向外侧。

4. 继续顺时针，在3点钟位置，六元环连接着一个苯环。苯环和中心环之间通过 -NH- 基团相连。

5. 在六元环的底部（6点钟位置），有一个较长的支链结构。这个支链由 -NH-CH2- 开始，然后连接一个五元环结构，可能是四氢呋喃环。

6. 在中心六元环的9点钟位置，有一个未标记的原子，可能是氮原子，构成嘧啶环结构。

7. 整个分子结构呈现出不对称的形状，主要由中心的杂环和三个不同的取代基组成。

8. 所有的化学键都以直线表示，没有显示任何立体化学信息。

9. 分子的左上方和右下方延伸得最远，而左下方相对较短。

这些描述应该能帮助OCR系统识别和定位分子结构中的各个部分。
``` 

实验:

  - 取500张图片, 进行第一次训练
  - 对这500张图片进行标记后, 进行第二次训练
  - 比较两次的效果

# 500张图片训练

标记前图片的配置文件: 

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
preprocessing_num_workers: 16

### output
output_dir: saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles.image_500
save_steps: 100
plot_loss: true

### train
per_device_train_batch_size: 8
gradient_accumulation_steps: 8
learning_rate: 5.0e-5
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

### special
max_samples: 500
warmup_steps: 0
lora_dropout: 0.2
weight_decay: 0.01
report_to: tensorboard
logging_dir: tensorboard_logging/image_500
logging_steps: 5
eval_steps: 5
num_train_epochs: 30.0

``` 

标记后图片的配置文件: 

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
preprocessing_num_workers: 16

### output
output_dir: saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles.image_500
save_steps: 100
plot_loss: true

### train
per_device_train_batch_size: 4
gradient_accumulation_steps: 8
learning_rate: 5.0e-5
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

### special
max_samples: 500
warmup_steps: 0
lora_dropout: 0.2
weight_decay: 0.01
report_to: tensorboard
logging_dir: tensorboard_logging/image_500
logging_steps: 5
eval_steps: 5
num_train_epochs: 30.0
``` 

需要将per_device_train_batch_size降低, 否则显存不足

命令: 

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0,1 NCCL_P2P_DISABLE=1 NCCL_IB_DISABLE=1 HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 \
llamafactory-cli train config/image_500.yaml" > train.log 2>&1 &
``` 

![image2024-10-31 21:24:38.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-10-31%2021%3A24%3A38.png)

从效果上: 

  - 比起没有描述的数据, 有描述的数据loss下降更快
  - 可能由于训练数据量小, 导致很快出现了过拟合

下一步: 用更多的数据进行训练, 扩大到5000张

# 5000张图片训练

![image2024-11-1 21:6:12.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-1%2021%3A6%3A12.png)

是否添加描述, loss曲线完全一致, 好像描述并没有用途.

调大了cutoff以后, 曲线并没有变化, 也就是说并不是因为截断引起的问题. 

需要诊断曲线完全一致的原因

去掉图片, 仅通过描述进行训练: 

![image2024-11-2 1:32:12.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-2%201%3A32%3A12.png)

曲线会比 只使用图片时 loss要高.

也就是说增加图片以后, 模型只尊重图片? 

去掉max_grad_norm (梯度裁剪), 曲线仍然没有改善

将Top-p设置为0.1, Temperature设置为0.01, 测试有描述和没有描述时的输出, 经过AB测试, 输出会稳定, 且有描述时的输出 不同于 没有描述时.

也就是说描述是有用的.

增大learning_rate: 1.0e-4, 训练loss会有下降, 但曲线趋势类同. LR下降较快

![image2024-11-2 22:6:28.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-2%2022%3A6%3A28.png)![image2024-11-2 22:6:41.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-2%2022%3A6%3A41.png)

增大learning_rate: 1.0e-3, loss会下降很多: 

![image2024-11-2 23:31:36.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-2%2023%3A31%3A36.png)

从实际测试效果上来说: 提供完整的提示词, 和 提供部分提示词相比, 效果没有差异. 比如: 

完整提示词: 

```
你的任务是分析图片和图片描述, 将其转换为SMILES标记。图片描述内容在<Description></Description>标签中(请注意这些描述可能不完全准确)。
<Description>
这张图片展示了一个复杂的有机分子结构。我将从上到下、从左到右详细描述这个结构:

1. 在分子的左上方有一个苯环结构。

2. 苯环的右上方连接着一个SO2基团,其中S原子位于中心,两个O原子分别位于上方和右方。

3. SO2基团的下方与苯环相连的是一个五元杂环,包含一个N原子。

4. 这个五元杂环的右侧连接着另一个N原子,该N原子再向右连接一个H原子。

5. 中间N原子的下方连接着一个C=O基团。

6. C=O基团的左侧连接着一个五元含硫杂环(噻吩环)。

7. 噻吩环的下方连接着一个六元环(环己烷),其中右下角有一个甲基(CH3)。

8. 回到中间的C=O基团,其右侧是一个O原子,再向右连接一个CH2CH3基团(乙氧基)。

这个分子结构包含了苯环、含氮五元环、噻吩环和环己烷等多种环状结构,以及SO2、C=O等官能团,整体呈现出一个复杂的有机分子骨架。
</Description>
图片如下: 
<image>
``` 

部分提示词: 

```
你的任务是分析图片和图片描述, 将其转换为SMILES标记。图片描述内容在<Description></Description>标签中(请注意这些描述可能不完全准确)。
<Description>
这张图片展示了一个复杂的有机分子结构。
这个分子结构包含了苯环、含氮五元环、噻吩环和环己烷等多种环状结构,以及SO2、C=O等官能团,整体呈现出一个复杂的有机分子骨架。
</Description>
图片如下: 
<image>
``` 

这两者产生的输出是一样的, 也就是说图片提供的信息 远胜于 文字提供的信息?

# 建立基线

5000张图片, LR=5e-2, loss下降缓慢

![image2024-11-3 10:49:34.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2010%3A49%3A34.png)

LR=5e-3, loss下降缓慢

![image2024-11-3 11:15:49.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2011%3A15%3A49.png)

LR=1e-3, loss下降还不错

![image2024-11-3 11:42:28.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2011%3A42%3A28.png)![image2024-11-3 11:42:49.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2011%3A42%3A49.png)

LR=1e-3, 更换LR策略为cosine_with_restarts, 尝试:

(配置文件:

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
preprocessing_num_workers: 16

### output
output_dir: saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles.image_5000
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
max_samples: 5000
warmup_steps: 0
lora_dropout: 0.2
weight_decay: 0.01
report_to: tensorboard
logging_dir: tensorboard_logging/image_5000
logging_steps: 20
eval_steps: 20
num_train_epochs: 8.0
learning_rate: 1.0e-3
lr_scheduler_type: cosine_with_restarts
lr_scheduler_kwargs:
    num_cycles: 2

``` 

)

![image2024-11-3 18:12:8.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2018%3A12%3A8.png)![image2024-11-3 18:12:17.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2018%3A12%3A17.png)

可以看到明确的LR重启

用同样的配置, 训练image_with_desc:

![image2024-11-3 21:9:5.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2021%3A9%3A5.png)![image2024-11-3 21:9:21.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2021%3A9%3A21.png)

loss曲线与image完全相同, 还是好像文字没有作用

仅使用描述(不是用图片)进行训练:

![image2024-11-3 22:19:39.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2022%3A19%3A39.png)![image2024-11-3 22:19:54.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2022%3A19%3A54.png)

loss变高, 但训练趋势相同. 是因为模型内的特征? 还是因为描述和图片高度重合?

使用完全空的描述进行训练:

![image2024-11-3 23:19:44.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-3%2023%3A19%3A44.png)

趋势与之前完全一样, 也就是说输入仅带来了部分信息量, 但没有改变loss的训练趋势??

检查输入:

样例: 

```
training example:
input_ids:
[151644, 8948, 198, 2610, 525, 264, 10950, 17847, 13, 151645, 198, 151644, 872, 198, 103929, 88802, 20412, 101042, 45930, 53481, 11, 58230, 228, 41146, 105359, 17714, 9501, 45978, 113743, 1773, 151645, 198, 151644, 77091, 198, 3706, 7612, 7, 28, 46, 46986, 16, 66, 8204, 34, 17, 28, 2448, 7, 28, 46, 2376, 28, 46, 46986, 18, 638, 37054, 18, 17, 8, 2388, 17, 66, 16, 53873, 3025, 8, 34, 17, 151645]
inputs:
<|im_start|>system
You are a helpful assistant.<|im_end|>
<|im_start|>user
你的任务是分析图片描述, 将其转换为SMILES标记。<|im_end|>
<|im_start|>assistant
CCOC(=O)c1c(NC2=NS(=O)(=O)c3ccccc32)sc2c1CCC(C)C2<|im_end|>
label_ids:
[-100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, -100, 3706, 7612, 7, 28, 46, 46986, 16, 66, 8204, 34, 17, 28, 2448, 7, 28, 46, 2376, 28, 46, 46986, 18, 638, 37054, 18, 17, 8, 2388, 17, 66, 16, 53873, 3025, 8, 34, 17, 151645]
labels:
CCOC(=O)c1c(NC2=NS(=O)(=O)c3ccccc32)sc2c1CCC(C)C2<|im_end|>
``` 

在/opt/huangyan/LLaMA-Factory/src/llamafactory/train/sft/trainer.py中, prediction_step中打印出inputs:

```
11/04/2024 00:22:52 - INFO - llamafactory.train.sft.trainer - TEST: input_ids: tensor([[151644,   8948,    198,   2610,    525,    264,  10950,  17847,     13,
         151645,    198, 151644,    872,    198, 103929,  88802,  20412, 101042,
          45930,  53481,     11,  58230,    228,  41146, 105359,  17714,   9501,
          45978, 113743,   1773, 151645,    198, 151644,  77091,    198,   8281,
             66,     16,  37054,   3759,      7,     28,     46,   2376,     28,
             46,  46986,     17,     66,   1016,     18,    638,  37054,     18,
             77,     17,      8,    638,     16, 151645, 151643, 151643, 151643,
         151643, 151643, 151643, 151643, 151643, 151643, 151643, 151643, 151643],
        [151644,   8948,    198,   2610,    525,    264,  10950,  17847,     13,
         151645,    198, 151644,    872,    198, 103929,  88802,  20412, 101042,
          45930,  53481,     11,  58230,    228,  41146, 105359,  17714,   9501,
          45978, 113743,   1773, 151645,    198, 151644,  77091,    198,   8281,
             66,     16,  37054,   8204,     34,      7,     28,     46,  46986,
             17,  37054,   4080,     66,     18,  37054,   2561,     45,     10,
           9533,     28,     46,   6620,     46,     12,   2467,    638,     18,
          88463,     17,      8,    638,     16, 151645, 151643, 151643, 151643]],
       device='cuda:0')
11/04/2024 00:22:52 - INFO - llamafactory.train.sft.trainer - TEST: input_ids: tensor([[151644,   8948,    198,   2610,    525,    264,  10950,  17847,     13,
         151645,    198, 151644,    872,    198, 103929,  88802,  20412, 101042,
          45930,  53481,     11,  58230,    228,  41146, 105359,  17714,   9501,
          45978, 113743,   1773, 151645,    198, 151644,  77091,    198,     46,
          40917,  19238,      8,   3706,     16,     46,   3706,  40917,     17,
          28668,     18,  53873,     19,     20,     34,     21,     28,   3706,
              7,     28,     46,      8,     34,      7,     28,     46,      8,
             34,   2561,     45,     10,   9533,     28,     46,   6620,     46,
             12,   2467,     28,     34,     21,   9949,     19,     34,     16,
             34,     17,   3706,     18,     20, 151645, 151643],
        [151644,   8948,    198,   2610,    525,    264,  10950,  17847,     13,
         151645,    198, 151644,    872,    198, 103929,  88802,  20412, 101042,
          45930,  53481,     11,  58230,    228,  41146, 105359,  17714,   9501,
          45978, 113743,   1773, 151645,    198, 151644,  77091,    198,  97754,
             16,  37054,  43644,      8,    638,     16,     45,  74071,      7,
             28,     46,      8,   9949,   3025,  46986,     16,    638,  37054,
             16,      8,     50,   3025,   2376,     28,     46,  11730,     46,
         151645, 151643, 151643, 151643, 151643, 151643, 151643, 151643, 151643,
         151643, 151643, 151643, 151643, 151643, 151643, 151643, 151643, 151643,
         151643, 151643, 151643, 151643, 151643, 151643, 151643]],
       device='cuda:1')
 
11/04/2024 00:22:52 - INFO - llamafactory.train.sft.trainer - TEST: attention_mask: tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0]],
       device='cuda:0')
11/04/2024 00:22:52 - INFO - llamafactory.train.sft.trainer - TEST: attention_mask: tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0],
        [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0,
         0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]], device='cuda:1')
 
11/04/2024 00:22:52 - INFO - llamafactory.train.sft.trainer - TEST: labels: tensor([[  -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   8281,
             66,     16,  37054,   3759,      7,     28,     46,   2376,     28,
             46,  46986,     17,     66,   1016,     18,    638,  37054,     18,
             77,     17,      8,    638,     16, 151645,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100],
        [  -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   8281,
             66,     16,  37054,   8204,     34,      7,     28,     46,  46986,
             17,  37054,   4080,     66,     18,  37054,   2561,     45,     10,
           9533,     28,     46,   6620,     46,     12,   2467,    638,     18,
          88463,     17,      8,    638,     16, 151645,   -100,   -100,   -100]],
       device='cuda:0')
11/04/2024 00:22:52 - INFO - llamafactory.train.sft.trainer - TEST: labels: tensor([[  -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,     46,
          40917,  19238,      8,   3706,     16,     46,   3706,  40917,     17,
          28668,     18,  53873,     19,     20,     34,     21,     28,   3706,
              7,     28,     46,      8,     34,      7,     28,     46,      8,
             34,   2561,     45,     10,   9533,     28,     46,   6620,     46,
             12,   2467,     28,     34,     21,   9949,     19,     34,     16,
             34,     17,   3706,     18,     20, 151645,   -100],
        [  -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,  97754,
             16,  37054,  43644,      8,    638,     16,     45,  74071,      7,
             28,     46,      8,   9949,   3025,  46986,     16,    638,  37054,
             16,      8,     50,   3025,   2376,     28,     46,  11730,     46,
         151645,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100]],
       device='cuda:1')
``` 

特殊字符含义: 

```
        151643: AddedToken("<|endoftext|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151644: AddedToken("<|im_start|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151645: AddedToken("<|im_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151646: AddedToken("<|object_ref_start|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151647: AddedToken("<|object_ref_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151648: AddedToken("<|box_start|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151649: AddedToken("<|box_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151650: AddedToken("<|quad_start|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151651: AddedToken("<|quad_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151652: AddedToken("<|vision_start|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151653: AddedToken("<|vision_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151654: AddedToken("<|vision_pad|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151655: AddedToken("<|image_pad|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
        151656: AddedToken("<|video_pad|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
``` 

其中并没有image的控制token, 其中文字部分包括prompt和labels ???

# 模型输入的调用堆栈

```
File "/opt/huangyan/LLaMA-Factory/src/llamafactory/launcher.py", line 23, in <module>
    launch()
  File "/opt/huangyan/LLaMA-Factory/src/llamafactory/launcher.py", line 19, in launch
    run_exp()
  File "/opt/huangyan/LLaMA-Factory/src/llamafactory/train/tuner.py", line 50, in run_exp
    run_sft(model_args, data_args, training_args, finetuning_args, generating_args, callbacks)
  File "/opt/huangyan/LLaMA-Factory/src/llamafactory/train/sft/workflow.py", line 96, in run_sft
    train_result = trainer.train(resume_from_checkpoint=training_args.resume_from_checkpoint)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/transformers/trainer.py", line 2052, in train
    return inner_training_loop(
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/transformers/trainer.py", line 2388, in _inner_training_loop
    tr_loss_step = self.training_step(model, inputs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/transformers/trainer.py", line 3485, in training_step
    loss = self.compute_loss(model, inputs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/transformers/trainer.py", line 3532, in compute_loss
    outputs = model(**inputs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/torch/nn/modules/module.py", line 1736, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/torch/nn/modules/module.py", line 1747, in _call_impl
    return forward_call(*args, **kwargs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/torch/nn/parallel/distributed.py", line 1643, in forward
    else self._run_ddp_forward(*inputs, **kwargs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/torch/nn/parallel/distributed.py", line 1459, in _run_ddp_forward
    return self.module(*inputs, **kwargs)  # type: ignore[index]
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/torch/nn/modules/module.py", line 1736, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/torch/nn/modules/module.py", line 1747, in _call_impl
    return forward_call(*args, **kwargs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/accelerate/utils/operations.py", line 820, in forward
    return model_forward(*args, **kwargs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/accelerate/utils/operations.py", line 808, in __call__
    return convert_to_fp32(self.model_forward(*args, **kwargs))
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/torch/amp/autocast_mode.py", line 44, in decorate_autocast
    return func(*args, **kwargs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/peft/peft_model.py", line 1577, in forward
    return self.base_model(
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/torch/nn/modules/module.py", line 1736, in _wrapped_call_impl
    return self._call_impl(*args, **kwargs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/torch/nn/modules/module.py", line 1747, in _call_impl
    return forward_call(*args, **kwargs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/peft/tuners/tuners_utils.py", line 188, in forward
    return self.model.forward(*args, **kwargs)
  File "/root/miniconda3/envs/huangyan-llama-factory/lib/python3.12/site-packages/transformers/models/qwen2_vl/modeling_qwen2_vl.py", line 1692, in forward
    tb_str = traceback.format_stack()
``` 

获取一次带图片调用的中间变量: trainer -> model -> 返回值. 发现inputs_embeds (input_ids)中已经包括了labels的内容?? 

```
11/04/2024 16:18:56 - INFO - llamafactory.train.sft.trainer - TEST: input_ids: tensor([[151644,   8948,    198,   2610,    525,    264,  10950,  17847,     13,
         151645,    198, 151644,    872,    198, 151652, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151653,  99650, 105518,  11319, 151645,    198, 151644,  77091,    198,
         110374, 100090, 102030, 101576,  99685,  56652,   9370, 104156, 100697,
          33108,  33983,  96465, 107630,  99603,   1773, 151645,    198, 151644,
            872,    198, 107469, 106428,  11319, 151645,    198, 151644,  77091,
            198, 107469, 102096, 109709, 106919,   1773, 151645, 151643, 151643,
         151643, 151643]], device='cuda:0')
11/04/2024 16:18:56 - INFO - llamafactory.train.sft.trainer - TEST: attention_mask: tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1, 1, 1, 1, 0, 0, 0, 0]], device='cuda:0')
11/04/2024 16:18:56 - INFO - llamafactory.train.sft.trainer - TEST: labels: tensor([[  -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
         110374, 100090, 102030, 101576,  99685,  56652,   9370, 104156, 100697,
          33108,  33983,  96465, 107630,  99603,   1773, 151645,   -100,   -100,
           -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,   -100,
           -100, 107469, 102096, 109709, 106919,   1773, 151645,   -100,   -100,
           -100,   -100]], device='cuda:0')
11/04/2024 16:18:56 - INFO - llamafactory.train.sft.trainer - TEST: pixel_values: tensor([[-1.3689, -1.2813, -1.1499,  ..., -1.2243, -1.1105, -1.0252],
        [-0.6536, -0.5660, -0.6244,  ..., -1.2100, -1.2100, -1.1958],
        [-1.7631, -1.7631, -1.7631,  ..., -1.4518, -1.4660, -1.4518],
        ...,
        [ 1.6238,  1.6238,  1.6092,  ...,  2.1459,  2.1459,  2.1459],
        [ 1.5946,  1.5946,  1.5946,  ...,  2.1175,  2.1032,  2.1032],
        [ 1.5800,  1.5800,  1.5654,  ...,  2.1317,  2.1317,  2.1317]],
       device='cuda:0')
11/04/2024 16:18:56 - INFO - llamafactory.train.sft.trainer - TEST: image_grid_thw: tensor([[ 1, 12, 22]], device='cuda:0')
[WARNING|modeling_qwen2_vl.py:1689] 2024-11-04 16:18:56,646 >> TEST: Qwen2VLForConditionalGeneration.forward: before image embed: inputs_embeds=tensor([[[-0.0104, -0.0011,  0.0062,  ...,  0.0025,  0.0030, -0.0001],
         [-0.0045, -0.0096,  0.0128,  ..., -0.0251, -0.0075, -0.0032],
         [ 0.0010,  0.0021, -0.0042,  ..., -0.0015,  0.0022, -0.0048],
         ...,
         [-0.0084,  0.0278,  0.0075,  ...,  0.0135, -0.0102, -0.0013],
         [-0.0084,  0.0278,  0.0075,  ...,  0.0135, -0.0102, -0.0013],
         [-0.0084,  0.0278,  0.0075,  ...,  0.0135, -0.0102, -0.0013]]],
       device='cuda:0', dtype=torch.bfloat16, requires_grad=True)
[WARNING|modeling_qwen2_vl.py:1690] 2024-11-04 16:18:56,647 >> TEST: Qwen2VLForConditionalGeneration.forward: before image embed: pixel_values=tensor([[-1.3689, -1.2813, -1.1499,  ..., -1.2243, -1.1105, -1.0252],
        [-0.6536, -0.5660, -0.6244,  ..., -1.2100, -1.2100, -1.1958],
        [-1.7631, -1.7631, -1.7631,  ..., -1.4518, -1.4660, -1.4518],
        ...,
        [ 1.6238,  1.6238,  1.6092,  ...,  2.1459,  2.1459,  2.1459],
        [ 1.5946,  1.5946,  1.5946,  ...,  2.1175,  2.1032,  2.1032],
        [ 1.5800,  1.5800,  1.5654,  ...,  2.1317,  2.1317,  2.1317]],
       device='cuda:0')
[WARNING|modeling_qwen2_vl.py:1724] 2024-11-04 16:18:57,379 >> TEST: Qwen2VLForConditionalGeneration.forward: input_ids=tensor([[151644,   8948,    198,   2610,    525,    264,  10950,  17847,     13,
         151645,    198, 151644,    872,    198, 151652, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655, 151655,
         151653,  99650, 105518,  11319, 151645,    198, 151644,  77091,    198,
         110374, 100090, 102030, 101576,  99685,  56652,   9370, 104156, 100697,
          33108,  33983,  96465, 107630,  99603,   1773, 151645,    198, 151644,
            872,    198, 107469, 106428,  11319, 151645,    198, 151644,  77091,
            198, 107469, 102096, 109709, 106919,   1773, 151645, 151643, 151643,
         151643, 151643]], device='cuda:0') outputs=BaseModelOutputWithPast(last_hidden_state=tensor([[[-0.7617,  0.5664, -1.2266,  ...,  0.4023,  0.5820, -0.1934],
         [-0.4902,  0.8906,  1.4453,  ...,  0.5078, -0.6367,  5.5625],
         [-2.6094,  2.3906,  1.8906,  ...,  0.7422, -0.8594,  1.8750],
         ...,
         [ 3.2656,  3.4062,  5.0938,  ...,  1.0625, -3.4375,  8.3750],
         [ 2.8438,  3.4531, -1.6641,  ...,  2.2344,  2.7344,  7.3125],
         [-0.9375,  6.6562,  0.2178,  ...,  1.8672, -1.1875,  3.1406]]],
       device='cuda:0', dtype=torch.bfloat16), past_key_values=None, hidden_states=None, attentions=None)
[WARNING|modeling_qwen2_vl.py:1725] 2024-11-04 16:18:57,382 >> TEST: Qwen2VLForConditionalGeneration.forward: inputs_embeds=tensor([[[-0.0104, -0.0011,  0.0062,  ...,  0.0025,  0.0030, -0.0001],
         [-0.0045, -0.0096,  0.0128,  ..., -0.0251, -0.0075, -0.0032],
         [ 0.0010,  0.0021, -0.0042,  ..., -0.0015,  0.0022, -0.0048],
         ...,
         [-0.0084,  0.0278,  0.0075,  ...,  0.0135, -0.0102, -0.0013],
         [-0.0084,  0.0278,  0.0075,  ...,  0.0135, -0.0102, -0.0013],
         [-0.0084,  0.0278,  0.0075,  ...,  0.0135, -0.0102, -0.0013]]],
       device='cuda:0', dtype=torch.bfloat16) outputs=BaseModelOutputWithPast(last_hidden_state=tensor([[[-0.7617,  0.5664, -1.2266,  ...,  0.4023,  0.5820, -0.1934],
         [-0.4902,  0.8906,  1.4453,  ...,  0.5078, -0.6367,  5.5625],
         [-2.6094,  2.3906,  1.8906,  ...,  0.7422, -0.8594,  1.8750],
         ...,
         [ 3.2656,  3.4062,  5.0938,  ...,  1.0625, -3.4375,  8.3750],
         [ 2.8438,  3.4531, -1.6641,  ...,  2.2344,  2.7344,  7.3125],
         [-0.9375,  6.6562,  0.2178,  ...,  1.8672, -1.1875,  3.1406]]],
       device='cuda:0', dtype=torch.bfloat16), past_key_values=None, hidden_states=None, attentions=None)
``` 

# 为什么模型训练时, input_ids中包含了labels的内容 (输入中包含了提示词+输出)

魔性训练使用了 Teacher Forcing 技术, 也就是说: 每步进行loss计算时, 都使用 "正确的"前置信息 推理 下一位的token; 而不是使用 推理出的前置信息 来推理 下一位的token (以防损耗扩大)

查看Qwen2VLForConditionalGeneration.forward, 其中的loss计算代码: 

```
        if labels is not None:
            # Shift so that tokens < n predict n
            shift_logits = logits[..., :-1, :].contiguous()
            shift_labels = labels[..., 1:].contiguous()
            # Flatten the tokens
            loss_fct = CrossEntropyLoss()
            shift_logits = shift_logits.view(-1, self.config.vocab_size)
            shift_labels = shift_labels.view(-1)
            # Enable model parallelism
            shift_labels = shift_labels.to(shift_logits.device)
            loss = loss_fct(shift_logits, shift_labels)
``` 

在loss_fct的调用中, 使用了 shift_logits 来推理 下一token, 与shift_labels进行对比. 

举例: 

输入 = [提示词Token-1, 提示词Token-2, 提示词Token-3, 理想输出Token-1, 理想输出Token-2, , 理想输出Token-3]

shift_logits = [提示词Token-1的logits, 提示词Token-2的logits, 提示词Token-3的logits, 理想输出Token-1的logits, 理想输出Token-2的logits]

shift_labels = [提示词Token-2的label, 提示词Token-3的label, 理想输出Token-1的label, 理想输出Token-2的label, 理想输出Token-3的label]

计算loss时, 举例: 使用 "提示词Token-3的logits" 推理下一token, 理想解是 "理想输出Token-1的label"

TODO:

```
如果使用了Teacher Forcing, 即使我的训练数据中的输入是完全空的 (输出不为空), 那么这个模型也可以进行训练, 并正确地通过输出的前半部分推理输出的后半部分. 
这跟真实使用的情况会有出入?
 
---
 
你的理解基本正确，但有一些细微的差别和需要注意的点。

**Teacher Forcing 在空输入下的训练：**

即使训练数据中的输入是完全空的，如果输出不为空，并且使用了 Teacher Forcing，模型仍然可以进行训练。这是因为 Teacher Forcing  **不依赖于模型自身的先前预测**。在每个时间步，模型都会接收真实的先前 token 作为输入，并尝试预测下一个 token。  即使输入为空，模型仍然可以学习从真实的输出序列中学习 token 之间的依赖关系和模式。

**推理过程与真实使用情况的出入：**

在推理过程中，模型需要根据自身的先前预测来生成后续的输出。 这与训练过程中使用 Teacher Forcing 的情况不同。 这种差异会导致以下问题：

* **Exposure Bias (暴露偏差):**  在训练过程中，模型总是接收真实的先前 token 作为输入。  但在推理过程中，模型需要依赖自身的先前预测，而这些预测可能包含错误。  这种训练和推理之间的差异被称为 Exposure Bias。  它会导致模型在推理过程中更容易积累错误，尤其是在生成长序列时。

* **不稳定的生成结果:**  由于模型在推理过程中依赖自身的先前预测，如果模型在某个时间步产生了错误的预测，这个错误可能会在后续时间步中被放大，导致生成结果不稳定。

**如何减轻这种出入：**

为了减轻 Teacher Forcing 带来的 Exposure Bias 和不稳定性，可以采用以下方法：

* **Scheduled Sampling (计划采样):**  在训练过程中，逐渐增加使用模型自身预测的概率，减少使用真实 token 的概率。  这可以帮助模型适应在推理过程中依赖自身预测的情况。

* **Professor Forcing:**  这是一种更高级的技术，它试图使模型在训练和推理过程中的隐藏状态分布更加一致，从而减少 Exposure Bias。

* **Beam Search (束搜索):**  在推理过程中，使用 Beam Search 可以探索多个可能的生成路径，并选择最优的路径。  这可以提高生成结果的质量和稳定性。

**总结：**

虽然在空输入的情况下，使用 Teacher Forcing 仍然可以训练模型，但这与真实使用情况存在出入。  Exposure Bias 和生成结果的不稳定性是需要关注的问题。  采用 Scheduled Sampling、Professor Forcing 和 Beam Search 等技术可以帮助减轻这些问题。
``` 

# 为什么输入为空时, 仍然loss会下降

习得了输出文本的序列关系

```
感谢您提供更多关于场景 4 的信息。根据您的描述，场景 4 的训练数据是将空字符串映射到不同的目标文本。在这种情况下，模型仍然可以学习，并且 loss 仍然可以下降，但这并非毫无意义的学习。

模型实际上在学习将空字符串作为一种特殊的输入，并根据这个输入预测不同的目标文本。可以理解为模型在学习一个基于空字符串的条件概率分布 P(目标文本 | 空字符串)。

虽然输入是空字符串，但模型仍然可以利用以下信息进行学习：

1. **目标文本的统计规律:** 模型可以学习目标文本中字符或单词的出现频率、共现关系等统计规律。例如，如果“结果”这个词频繁出现在目标文本中，模型可能会学习到在输入为空字符串时，输出应该包含“结果”这个词。

2. **目标文本的序列信息:**  如果目标文本具有一定的序列规律（例如语法结构、逻辑关系），模型可以学习这些序列信息，并在生成文本时体现出来。

3. **模型参数:**  即使输入为空字符串，模型的参数仍然会在训练过程中不断调整，以更好地拟合目标文本的分布。

在这种情况下，loss 的下降表明模型正在学习如何根据空字符串生成更符合训练数据分布的目标文本。

然而，这种学习方式的泛化能力可能比较有限。因为模型仅仅依赖于目标文本自身的规律，而没有来自输入的指导信息。当遇到新的、未在训练数据中出现过的目标文本时，模型的预测能力可能会下降。

为了提高模型的泛化能力，可以考虑以下方法：

1. **引入额外的输入信息:**  即使无法提供与目标文本直接相关的输入，也可以尝试引入一些通用的背景信息，例如时间、地点、主题等，作为模型的输入。

2. **使用更强大的模型结构:**  例如 Transformer 模型，可以更好地捕捉长距离的序列依赖关系，从而提高生成文本的质量。

3. **改进训练策略:**  例如使用 curriculum learning，先训练简单的目标文本，再逐渐增加难度，可以帮助模型更好地学习。

总而言之，场景 4 的训练并非毫无意义，模型正在学习如何根据空字符串生成目标文本。但是，由于缺乏输入信息的指导，模型的泛化能力可能有限。为了提高模型的性能，需要考虑引入额外的输入信息或改进模型结构和训练策略。
``` 

# 对于Loss形状相似的猜测

曲线一致但高低不同, 意味着: 

  - 输入是有用的, 可以帮助模型确定 大概 用哪一组输出
  - 模型在训练过程中, 更多地捕捉了 输出序列的规律性, 而不是通过输入来影响输出的某一部分

猜测是因为输出是smiles表达式, smiles表达式在tokenizer阶段生成的token的语义性较弱, 导致模型会更多学习smiles表达式的内部规律

```
问题描述: 
 
我在训练一个多模大模型, 我测试了以下四种场景: 
场景1: 使用图片+描述作为输入
场景2: 使用图片作为输入
场景3: 使用描述作为输入
场景4: 使用空字符串描述作为输入

我发现四种场景中, loss的曲线趋势是完全相同的, 其loss曲线的大小关系是 4>3>2=1.

---
 
虽然两个场景的 loss 曲线形状一致，但场景 3（文本描述输入）的 loss 曲线比场景 4（空字符串输入）低，这说明**模型在一定程度上利用了输入的文本描述信息**，尽管这种利用可能不够充分，正如我们之前讨论的，模型可能仍然主要依赖于输出序列的自身规律性。

Loss 更低意味着模型在场景 3 下能够更好地预测目标文本，这可以归因于输入文本描述提供了一些有用的信息。 这些信息帮助模型缩小了预测范围，使其更容易找到正确的目标文本。  可以想象一下，如果没有输入文本（场景 4），模型需要从所有可能的文本序列中进行搜索；而有了输入文本（场景 3），模型的搜索空间就缩小了，因此更容易找到正确的答案，从而降低了 loss。

然而，由于 loss 曲线形状相似，这仍然暗示模型对输入信息的利用可能不够充分，或者模型结构本身更偏向于捕捉输出序列的规律性。  这就像模型得到了一点提示（输入文本），但并没有完全理解和利用这个提示，只是稍微借鉴了一下。

为了更深入地理解这一点，您可以结合之前的建议，进行以下分析：

* **量化输入信息的影响:**  尝试计算在场景 3 中，输入文本描述对模型预测的贡献程度。  这可以通过一些解释性方法来实现，例如分析注意力权重，或者通过扰动输入文本观察输出的变化。

* **分析不同类型输入文本的影响:**  尝试使用不同类型的输入文本描述进行实验，例如更长、更短、更具体或更抽象的描述。观察 loss 的变化，以及模型输出质量的变化。  这可以帮助您了解模型对不同类型输入信息的敏感程度。

* **比较不同模型结构的性能:**  尝试使用不同的模型结构，例如更复杂的编码器或更强的注意力机制。比较它们在场景 3 和场景 4 下的性能差异。如果某个模型在场景 3 下的性能提升更显著，则说明该模型能够更好地利用输入信息。

总而言之，场景 3 的 loss 低于场景 4 说明模型并非完全忽略了输入信息，但 loss 曲线形状相似又表明模型对输入信息的利用可能不够充分。需要进一步的分析和实验来确定问题的根源，并找到改进模型的方法。  目标是让模型更有效地利用输入信息，从而在场景 3 下取得更大的性能提升，并且使场景 3 和场景 4 的 loss 曲线形状出现差异。
``` 

寻找一个支持smiles的tokenizer? 

  - <https://github.com/rxn4chemistry/rxnfp/blob/master/rxnfp/tokenization.py>
  - 尝试用selfies格式? 

# 尝试用image->selfies格式是否能缓解loss曲线一致的问题

使用python的selfies库, 将smiles表达式转成selfies表达式, 对比"没有描述"和"仅用文字描述"两个场景下的loss曲线. 

如果曲线走势不同, 可证明 selfies格式 能让大模型对输出理解更好, 使得输入更多地参与到了推理中.

如果走势相同, 无法得出新的结论. 

![image2024-11-5 16:39:9.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-5%2016%3A39%3A9.png)

结论: 曲线相似

# 尝试用smiles的语法树来代替smiles表达式, 提供更多的语法信息, 缓解loss曲线一致的问题

[smiles的grammer文件: https://github.com/antlr/grammars-v4/tree/master/smiles](<https://github.com/antlr/grammars-v4/tree/master/smiles>)

使用antlr生成python的antlr处理代码: 

```
apt install antlr4
pip install antlr4-python3-runtime==4.7.2
 
``` 

注意以上runtime的版本, 需要和apt安装的antlr4的java jar的版本一致, 否则会报错: Exception: Could not deserialize ATN with version 3 (expected 4).

降低LR=1e-4: 

```
### model
model_name_or_path: Qwen/Qwen2-VL-7B-Instruct

### method
stage: sft
do_train: true
finetuning_type: lora
lora_target: all

### dataset
dataset: decimer-img2smiles_antlr4-only_desc
dataset_dir: data
template: qwen2_vl
cutoff_len: 1024
preprocessing_num_workers: 16

### output
output_dir: saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2smiles_antlr4.only_desc_5000
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
max_samples: 5000
warmup_steps: 0
lora_dropout: 0.2
weight_decay: 0.01
report_to: tensorboard
logging_dir: tensorboard_logging/img2smiles_antlr4_only_desc_5000
logging_steps: 20
eval_steps: 20
num_train_epochs: 8.0
learning_rate: 1.0e-4
lr_scheduler_type: cosine_with_restarts
lr_scheduler_kwargs:
    num_cycles: 2

``` 

用少量step进行测试, 两条loss曲线形状会有略微区别?? 需要扩大step, 并继续缩小LR到5e-5, lr_scheduler_type改成cosine (cosine_with_restarts用处不大), 缩小epoch

![image2024-11-5 23:49:54.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-5%2023%3A49%3A54.png)

进行完整的测试: 

![image2024-11-6 23:4:25.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-6%2023%3A4%3A25.png)

loss趋势: 空描述无图片 > 仅有描述 > 仅有图片 = 图片+描述

loss曲线趋势正常, 也就是说当输出为 Smiles-Antlr4语法树的形式时, 其输入能影响到输出.

惟一的问题是 仅有图片 和 图片+描述 两个场景中, 描述并没有起到作用. 有以下两种可能: 

  1. 描述是从claude-3.5-sonnet中生成的, 而Qwen2-VL模型本身已经具备了跟claude对图片相同的理解, 所以描述并不能有助于对图片的理解
  2. 当图片和描述同时存在时, Qwen2-VL给图片的权重过重, 导致描述无法影响输出结果

需要设计实验来进行证明

TODO:

尝试下调LR, 让模型学习细节试试:

![image2024-11-7 0:22:59.png](/assets/01KJBZJ68Z33PJAEYYR9H9S25Y/image2024-11-7%200%3A22%3A59.png)

loss曲线并不能变更好

# SMILES工具集

  - <https://www.antvaset.com/smiles-to-structure>
