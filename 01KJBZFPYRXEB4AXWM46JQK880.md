---
title: 20240926 - 使用 Llama-factory 做强化学习 [2]
confluence_page_id: 3342498
created_at: 2024-09-25T17:25:42+00:00
updated_at: 2024-10-03T15:03:03+00:00
---

# 选取数据集

reward: [https://huggingface.co/datasets/markyfsun/chinese-enthusiastic-dpo](<https://huggingface.co/datasets/markyfsun/chinese-enthusiastic-dpo?row=2>)

sft + PPO: Pistachio-LLM/llama3-8b-chinese-medqa

# 训练reward model

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage rm \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-enthusiastic-dpo \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 1.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 1 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/reward-chinese-enthusiastic-dpo \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --do_eval True \
    --evaluation_strategy steps \
    --eval_dataset chinese-enthusiastic-dpo-500-samples \
    --per_device_eval_batch_size 1 \
    --eval_steps 100 \
    --eval_on_start " > train.log 2>&1 &
``` 

在0.11轮次时, 准确率就已经到了0.988:

```
{'eval_loss': 0.740327775478363, 'eval_accuracy': 0.334, 'eval_runtime': 45.1631, 'eval_samples_per_second': 11.071, 'eval_steps_per_second': 11.071, 'epoch': 0, 'num_input_tokens_seen': 0}
{'eval_loss': 0.023381507024168968, 'eval_accuracy': 0.988, 'eval_runtime': 44.6709, 'eval_samples_per_second': 11.193, 'eval_steps_per_second': 11.193, 'epoch': 0.11, 'num_input_tokens_seen': 258032}
``` 

注意: 当前使用了epoch=1的训练结果, 其可能过拟合, 并没有测试其泛化性

# 微调

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-medqa \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 3.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 1 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/sft-chinese-medqa \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 100 \
    --per_device_eval_batch_size 1" > train.log 2>&1 &
``` 

epoch=3, 没有验证集:

![image2024-9-26 19:19:55.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-9-26%2019%3A19%3A55.png)

裸模型: 

![image2024-9-26 19:31:41.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-9-26%2019%3A31%3A41.png)

训练数据的效果: 

"孩子缺钙可能是导致没有长牙的原因，建议给孩子补钙。可以使用迪巧、鱼肝油等药物，连续服用一个月就会有明显的效果。同时，可以让孩子多晒太阳，多吃富含钙质的食物，如奶制品、豆制品、海产品等。"

微调后的效果: 

![image2024-9-26 19:35:12.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-9-26%2019%3A35%3A12.png)

# PPO训练

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage ppo \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-medqa \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 1.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 2 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/ppo-chinese-medqa \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --adapter_name_or_path saves/Qwen2-7B/lora/sft-chinese-medqa \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --reward_model saves/Qwen2-7B/lora/reward-chinese-enthusiastic-dpo \
    --reward_model_type lora \
    --top_k 0 \
    --top_p 0.9" > train.log 2>&1 &
``` 

![image2024-9-27 18:30:43.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-9-27%2018%3A30%3A43.png)

![image2024-9-27 18:30:49.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-9-27%2018%3A30%3A49.png)

实测效果: 对于中文只会返回空: 

![image2024-9-27 18:31:28.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-9-27%2018%3A31%3A28.png)

原因不清楚

使用代码进行模型加载和测试: [TestPpoModel.ipynb](/assets/01KJBZFPYRXEB4AXWM46JQK880/TestPpoModel.ipynb)

发现: sft_model能反应微调后的结果, 而ppo_model 和 raw_model的输出一样, 也就是ppo回滚了sft的效果??

在raw_model上, 直接加载ppo层 (而不是在sft层上加载ppo层), 发现输出变为空, 跟GUI的效果一样

诊断: 将sft和ppo模型的lora层都打印出来: [TestPpoModel-ppo-is-zero.ipynb](/assets/01KJBZFPYRXEB4AXWM46JQK880/TestPpoModel-ppo-is-zero.ipynb), 发现: 

PPO模型的lora层的lora_B矩阵都为空, 以上检查脚本的输出: 

```
PPO LoRA weights for base_model.model.base_model.model.model.layers.0.self_attn.q_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.0.self_attn.k_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.0.self_attn.v_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.0.self_attn.o_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.0.mlp.gate_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.0.mlp.up_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.0.mlp.down_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.1.self_attn.q_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.1.self_attn.k_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.1.self_attn.v_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.1.self_attn.o_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.1.mlp.gate_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.1.mlp.up_proj.lora_B.default.weight are all zeros.
PPO LoRA weights for base_model.model.base_model.model.model.layers.1.mlp.down_proj.lora_B.default.weight are all zeros.
``` 

尝试将sft合并到基模型里, 再进行PPO:

```
from peft import PeftModel, PeftConfig
sft_adapter_name = "saves/Qwen2-7B/lora/sft-chinese-medqa"
sft_model = PeftModel.from_pretrained(raw_model, sft_adapter_name)

sft_model = sft_model.merge_and_unload()
print("merged sft model")

sft_model.save_pretrained("saves/Qwen2-7B/lora/merged-sft-chinese-medqa")
print("saved sft model")
 
tokenizer.save_pretrained("saves/Qwen2-7B/lora/merged-sft-chinese-medqa")
print("saved tokenizer")

# reload
 
sft_model.to("cpu")
del sft_model

sft_model = AutoModelForCausalLM.from_pretrained(
    pretrained_model_name_or_path="saves/Qwen2-7B/lora/merged-sft-chinese-medqa", 
    config=AutoConfig.from_pretrained(
        "saves/Qwen2-7B/lora/merged-sft-chinese-medqa",
        trust_remote_code= True,
        revision= "main",
        compute_dtype=torch.bfloat16,
    ),
    torch_dtype="auto",
    trust_remote_code=True,
    low_cpu_mem_usage=True,
).to(device)
print(sft_model)
``` 

训练0.1个epoch, 看到PPO的lora层权重都不为0

训练1.0个epoch, 看到PPO的lora层权重都不为0, 但输出的结果全部为空

下一步: 在训练过程中, 每5个step保留checkpoint, 查看输出结果是否是突然消失: 

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage ppo \
    --do_train True \
    --model_name_or_path saves/Qwen2-7B/lora/merged-sft-chinese-medqa \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-medqa \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 1.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 2 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 5 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/ppo-chinese-medqa \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --reward_model saves/Qwen2-7B/lora/reward-chinese-enthusiastic-dpo \
    --reward_model_type lora \
    --top_k 0 \
    --top_p 0.9" > train.log 2>&1 &
``` 

发现 5/10/15 step时, 模型输出会减少至"空". 认为是reward模型的输出有问题, 变成了"鼓励"模型闭嘴

在PPO训练过程中, get_rewards函数中增加代码: 

```
print(f"get_rewards: reward={rewards_cpu}, queries.shape={[tensor.shape[0] for tensor in queries]}, responses.shape={[tensor.shape[0] for tensor in responses]}")

if len(rewards_cpu) == 2:
    rewards_relation = rewards_cpu[0] > rewards_cpu[1]
    responses_relation = responses_cpu[0] > responses_cpu[1]
    if rewards_relation == responses_relation:
        print("ok")
    else:
        print("bad")
``` 

打印的结果举例: 

```
get_rewards: reward=tensor([[29.6250],
        [34.5000]]), queries.shape=[61, 60], responses.shape=[19, 11]
bad
get_rewards: reward=tensor([[35.0000],
        [40.2500]]), queries.shape=[42, 65], responses.shape=[16, 25]
ok
get_rewards: reward=tensor([[44.2500],
        [28.3750]]), queries.shape=[62, 36], responses.shape=[5, 17]
bad
get_rewards: reward=tensor([[35.0000],
        [22.1250]]), queries.shape=[56, 117], responses.shape=[23, 10]
ok
``` 

对结果进行分析: 脚本: [11.py](/assets/01KJBZFPYRXEB4AXWM46JQK880/11.py), 结果: 

Reward 值和 Response 长度的相关系数: -0.30775957242124613

![image2024-9-29 19:59:8.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-9-29%2019%3A59%3A8.png)

使用一套独立的脚本, 将数据固定下来, 测试不同的reward model的表现. (来调整reward model的训练参数):

  - 独立测试脚本: [TestRewardModel_DataDistribution.ipynb](/assets/01KJBZFPYRXEB4AXWM46JQK880/TestRewardModel_DataDistribution.ipynb)
  - 数据: [rewards_data.pt](/assets/01KJBZFPYRXEB4AXWM46JQK880/rewards_data.pt)

现在用的reward model 的评估: 

  - Reward 值和 Response 长度的相关系数: -0.32023687331223055

  - ![image2024-9-30 23:22:56.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-9-30%2023%3A22%3A56.png)

对reward model的训练进行调整, epoch调整为0.1:

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage rm \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-enthusiastic-dpo \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 0.1 \
    --max_samples 100000 \
    --per_device_train_batch_size 1 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/reward-chinese-enthusiastic-dpo-2 \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --do_eval True \
    --evaluation_strategy steps \
    --eval_dataset chinese-enthusiastic-dpo-500-samples \
    --per_device_eval_batch_size 1 \
    --eval_steps 100 \
    --eval_on_start " > train.log 2>&1 &
``` 

训练后的评估数据: 

```
  epoch                   =     0.1006
  eval_accuracy           =      0.988
  eval_loss               =     0.0304
  eval_runtime            = 0:00:44.71
  eval_samples_per_second =     11.181
  eval_steps_per_second   =     11.181
  num_input_tokens_seen   =     225040
``` 

使用这个模型, 跑独立测试脚本:

  - Reward 值和 Response 长度的相关系数: 0.004979562674788149

  - ![image2024-9-30 23:38:54.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-9-30%2023%3A38%3A54.png)

  - 没有调整loss, 仅减少epoch, 正确率仍然比较高, 但与长度的相关性减弱很多.

使用这个模型, 对PPO进行训练, 扩大epoch到3.0:

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage ppo \
    --do_train True \
    --model_name_or_path saves/Qwen2-7B/lora/merged-sft-chinese-medqa \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-medqa \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 3.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 2 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 5 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/ppo-chinese-medqa \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --reward_model saves/Qwen2-7B/lora/reward-chinese-enthusiastic-dpo-2 \
    --reward_model_type lora \
    --top_k 0 \
    --top_p 0.9" > train.log 2>&1 &
``` 

抽查step=25时, PPO模型的输出: 

  - SFT: 我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据病情的严重程度来确定。如果病情较轻，可以采用药物治疗，如抗生素、消炎药等。如果病情较重，可能需要住院治疗，进行输液、吸氧等治疗。此外，家长还需要注意宝宝的饮食和休息，保持室内空气流通，避免宝宝感染其他疾病。
  - PPO: 我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据病情严重程度来确定。轻度肺炎可以通过药物治疗，如抗生素和抗病毒药物。如果病情严重，可能需要住院治疗，包括输液和氧气治疗。此外，保持宝宝的水分摄入和营养摄入也很重要。

epoch=3.0时, step=530:

  - ![image2024-10-1 11:53:48.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-1%2011%3A53%3A48.png)
  - ![image2024-10-1 11:53:56.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-1%2011%3A53%3A56.png)
  - SFT: 我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据病情的严重程度来确定。如果病情较轻，可以采用药物治疗，如抗生素、消炎药等。如果病情较重，可能需要住院治疗，进行输液、吸氧等治疗。此外，家长还需要注意宝宝的饮食和休息，保持室内空气流通，避免宝宝感染其他疾病。
  - PPO: 我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据病情的严重程度来确定。轻度的新生儿肺炎可以通过药物治疗来缓解症状，而严重的新生儿肺炎可能需要住院治疗。此外，新生儿肺炎的治疗还需要注意保持宝宝的水分摄入，避免宝宝脱水。

  - 看起来reward的特征并不明显

再次训练reward, 将训练集和校验集分开 (现在的校验集是训练集的子集):

分离训练集和校验集的代码: 

```
from datasets import load_dataset
import json

# 加载数据集
dataset = load_dataset("markyfsun/chinese-enthusiastic-dpo")

# 划分训练集和测试集
train_test_split = dataset['train'].train_test_split(test_size=0.1, seed=42)
train_set = train_test_split['train']
test_set = train_test_split['test']

# 保存为JSON文件
with open("data/chinese-enthusiastic-dpo_train.json", "w", encoding="utf-8") as f:
    json.dump(train_set.to_list(), f, ensure_ascii=False, indent=4)

with open("data/chinese-enthusiastic-dpo_eval.json", "w", encoding="utf-8") as f:
    json.dump(test_set.to_list(), f, ensure_ascii=False, indent=4)

# 输出记录数
print(f"训练集记录数: {len(train_set)}")
print(f"测试集记录数: {len(test_set)}")

# 训练集记录数: 6300
# 测试集记录数: 700
``` 

使用新数据集训练PPO:

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage rm \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-enthusiastic-dpo-train \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 3 \
    --max_samples 100000 \
    --per_device_train_batch_size 1 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/reward-chinese-enthusiastic-dpo-3 \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --do_eval True \
    --evaluation_strategy steps \
    --eval_dataset chinese-enthusiastic-dpo-eval \
    --per_device_eval_batch_size 1 \
    --eval_steps 100 \
    --eval_on_start " > train.log 2>&1 &
``` 

效果：

![image2024-10-1 16:39:26.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-1%2016%3A39%3A26.png)![image2024-10-1 16:39:33.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-1%2016%3A39%3A33.png)![image2024-10-1 16:39:46.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-1%2016%3A39%3A46.png)

epoch=3.0, step=2361, Reward 值和 Response 长度的相关系数: -0.1706594354506166

step-500, Reward 值和 Response 长度的相关系数: 0.032229425597057264

step-200, Reward 值和 Response 长度的相关系数: -0.05728302829693278

step-100, Reward 值和 Response 长度的相关系数: -0.031031033254542637

用step-500的reward model, 进行PPO训练

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage ppo \
    --do_train True \
    --model_name_or_path saves/Qwen2-7B/lora/merged-sft-chinese-medqa \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-medqa \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 1.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 2 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 5 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/ppo-chinese-medqa \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --reward_model saves/Qwen2-7B/lora/reward-chinese-enthusiastic-dpo-3/checkpoint-500 \
    --reward_model_type lora \
    --top_k 0 \
    --top_p 0.9" > train.log 2>&1 &
``` 

![image2024-10-2 0:12:15.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-2%200%3A12%3A15.png)![image2024-10-2 0:12:22.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-2%200%3A12%3A22.png)

回答中的输出又消失了

尝试用reward model (step = 200) 再进行一次PPO训练, 主要思路是 不让reward model过拟合, 再进行训练. 

![image2024-10-2 10:49:31.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-2%2010%3A49%3A31.png)![image2024-10-2 10:49:38.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-2%2010%3A49%3A38.png)

输出: 

SFT: 

我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据病情的严重程度来确定。如果病情较轻，可以采用药物治疗，如抗生素、消炎药等。如果病情较重，可能需要住院治疗，进行输液、吸氧等治疗。此外，家长还需要注意宝宝的饮食和休息，保持室内空气流通，避免宝宝感染其他疾病。 

PPO:

我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？你好！针对宝宝肺炎的治疗，可以考虑一些综合调理的方法来帮助恢复。同时，保持宝宝的饮食均衡也是很重要的哦！ 

在这次PPO训练中, reward model (step=200) 将"轻松"的语言习惯训练进去了, 但长度也越来越短

# 在reward model训练过程中, 增加长度损失惩罚

在rm/trainer.py中, 增加长度惩罚:

```
    @override
    def compute_loss(
        self, model: "PreTrainedModel", inputs: Dict[str, "torch.Tensor"], return_outputs: bool = False
    ) -> Union["torch.Tensor", Tuple["torch.Tensor", List["torch.Tensor"]]]:
        r"""
        Computes pairwise loss. The first n examples are chosen and the last n examples are rejected.

        Subclass and override to inject custom behavior.

        Note that the first element will be removed from the output tuple.
        See: https://github.com/huggingface/transformers/blob/v4.40.0/src/transformers/trainer.py#L3842
        """
        _, _, values = model(**inputs, output_hidden_states=True, return_dict=True, use_cache=False)
        batch_size = inputs["input_ids"].size(0) // 2
        chosen_masks, rejected_masks = torch.split(inputs["attention_mask"], batch_size, dim=0)
        chosen_rewards, rejected_rewards = torch.split(values, batch_size, dim=0)
        chosen_scores = chosen_rewards.gather(dim=-1, index=(chosen_masks.sum(dim=-1, keepdim=True) - 1))
        rejected_scores = rejected_rewards.gather(dim=-1, index=(rejected_masks.sum(dim=-1, keepdim=True) - 1))
        chosen_scores, rejected_scores = chosen_scores.squeeze(), rejected_scores.squeeze()

        loss = -torch.nn.functional.logsigmoid(chosen_scores.float() - rejected_scores.float()).mean()

        # # ---  新增部分：计算长度惩罚 ---
        chosen_lengths = chosen_masks.sum(dim=-1)
        rejected_lengths = rejected_masks.sum(dim=-1)
        length_difference = torch.abs(chosen_lengths - rejected_lengths)

        length_penalty_factor = 0.05 * loss.item()  # 使其与 loss 的数量级相关
        length_penalty = length_penalty_factor * length_difference.float().mean()
        loss = loss + length_penalty
        # print(f"compute_loss: loss={loss}, length_penalty={length_penalty}")
        # # ---  新增部分结束 ---

        if return_outputs:
            return loss, (loss, chosen_scores, rejected_scores)
        else:
            return loss
``` 

训练RM: 

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage rm \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-enthusiastic-dpo-train \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 3 \
    --max_samples 100000 \
    --per_device_train_batch_size 1 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/reward-chinese-enthusiastic-dpo-4 \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --do_eval True \
    --evaluation_strategy steps \
    --eval_dataset chinese-enthusiastic-dpo-eval \
    --per_device_eval_batch_size 1 \
    --eval_steps 100 \
    --eval_on_start " > train.log 2>&1 &
``` 

![image2024-10-2 22:35:0.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-2%2022%3A35%3A0.png)![image2024-10-2 22:35:6.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-2%2022%3A35%3A6.png)![image2024-10-2 22:35:14.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-2%2022%3A35%3A14.png)

用step=500的RM 训练PPO:

```
rm -rf saves/Qwen2-7B/lora/ppo-chinese-medqa
 
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage ppo \
    --do_train True \
    --model_name_or_path saves/Qwen2-7B/lora/merged-sft-chinese-medqa \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-medqa \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 1.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 2 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 5 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/ppo-chinese-medqa \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --reward_model saves/Qwen2-7B/lora/reward-chinese-enthusiastic-dpo-4/checkpoint-500 \
    --reward_model_type lora \
    --top_k 0 \
    --top_p 0.9" > train.log 2>&1 &
``` 

训练中内存超出, 仅训练了70个step, 测试一下, 看起来长度惩罚已经起了作用: 

```
SFT:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据病情的严重程度来确定。如果病情较轻，可以采用药物治疗，如抗生素、消炎药等。如果病情较重，可能需要住院治疗，进行输液、吸氧等治疗。此外，家长还需要注意宝宝的饮食和休息，保持室内空气流通，避免宝宝感染其他疾病。

PPO-10:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据病情的严重程度来确定。一般来说，轻度的肺炎可以通过口服抗生素来治疗，而严重的肺炎可能需要住院治疗，使用静脉注射抗生素。此外，还需要注意宝宝的饮食和休息，保持室内空气流通，避免宝宝接触烟雾和有害气体。如果宝宝出现呼吸急促、发热、咳嗽、食欲不振等症状，应及时就医。祝宝宝早日康复！

PPO-30:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据病情严重程度来确定。一般来说，轻度肺炎可以通过口服抗生素治疗，而严重肺炎可能需要住院治疗。此外，还需要注意宝宝的饮食和休息，保持室内空气流通，避免感染。祝宝宝早日康复！

PPO-40:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据病情严重程度来确定。一般来说，轻度肺炎可以通过口服抗生素来治疗，而重度肺炎可能需要住院治疗。此外，还需要注意宝宝的饮食和休息，保持室内空气流通，避免感染。建议您咨询专业医生，根据宝宝的具体情况制定治疗方案。祝宝宝早日康复！

PPO-50:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据孩子的具体情况来确定。一般来说，治疗方案包括抗生素、支气管扩张剂、氧气治疗等。此外，还需要注意孩子的饮食和休息，保持良好的生活习惯。如果孩子出现发热、咳嗽、呼吸困难等症状，应及时就医。希望孩子早日康复！

PPO-60:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎是一种常见的疾病，需要及时治疗。治疗方案包括抗生素、支持治疗和对症治疗。抗生素是治疗肺炎的主要药物，可以根据孩子的具体情况选择合适的抗生素。支持治疗包括保持孩子的营养和水分摄入，以及保持孩子的体温稳定。对症治疗包括缓解孩子的症状，如咳嗽、呼吸困难和发热等。治疗方案需要根据孩子的具体情况制定，建议您咨询专业的医生，根据孩子的具体情况制定治疗方案。祝您的孩子早日康复！

PPO-70:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据孩子的具体情况来确定。一般来说，医生会根据孩子的年龄、体重、病情严重程度等因素来制定治疗方案。治疗方案包括药物治疗和非药物治疗。药物治疗包括抗生素、支气管扩张剂、抗病毒药物等。非药物治疗包括吸氧、雾化、物理治疗等。治疗期间，家长需要密切观察孩子的病情变化，及时调整治疗方案。同时，家长还需要注意孩子的饮食和休息，保持良好的生活习惯，增强孩子的免疫力，促进孩子的康复。希望孩子早日康复！

``` 

将per_device_train_batch_size调成1:

```
rm -rf saves/Qwen2-7B/lora/ppo-chinese-medqa
 
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 \
llamafactory-cli train \
    --stage ppo \
    --do_train True \
    --model_name_or_path saves/Qwen2-7B/lora/merged-sft-chinese-medqa \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset chinese-medqa \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 1.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 1 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 5 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/ppo-chinese-medqa \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --reward_model saves/Qwen2-7B/lora/reward-chinese-enthusiastic-dpo-4/checkpoint-500 \
    --reward_model_type lora \
    --top_k 0 \
    --top_p 0.9" > train.log 2>&1 &
``` 

训练效果: 

![image2024-10-3 22:56:46.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-3%2022%3A56%3A46.png)![image2024-10-3 22:56:51.png](/assets/01KJBZFPYRXEB4AXWM46JQK880/image2024-10-3%2022%3A56%3A51.png)

文本会越来越长, 然后越来越短?? 

但已经习得了语言风格

```
SFT:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎的治疗需要根据病情的严重程度来确定。如果病情较轻，可以采用药物治疗，如抗生素、消炎药等。如果病情较重，可能需要住院治疗，进行输液、吸氧等治疗。此外，家长还需要注意宝宝的饮食和休息，保持室内空气流通，避免宝宝感染其他疾病。
 
PPO-50:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎是一种常见的疾病，需要及时治疗。治疗方案包括抗生素、氧气治疗、支持性治疗等。同时，家长需要注意宝宝的饮食和休息，保持室内空气清新，避免感染传播。祝宝宝早日康复！

PPO-75:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎是一种常见的疾病，需要及时治疗。治疗方案会根据孩子的具体情况和医生的建议而定。一般来说，医生会给孩子开具适当的药物治疗，如抗生素、咳嗽药等。同时，家长也需要注意孩子的饮食和休息，保证他们的营养和休息充足，增强免疫力，促进康复。希望宝宝早日康复，健康成长！

PPO-90:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎是一种常见的疾病，需要及时治疗，否则会对孩子的健康造成严重影响。治疗新生儿肺炎的方法有很多种，具体治疗方案需要根据孩子的具体情况和医生的建议来确定。一般来说，医生会根据孩子的年龄、体重、病情严重程度等因素，选择合适的药物治疗方案，如抗生素、抗病毒药物等。同时，医生也会给予孩子适当的护理和照顾，保证孩子的营养和休息，促进康复。希望宝宝能够早日康复，健康成长！

PPO-100:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎是一种常见的疾病，需要及时治疗，否则会对孩子的健康造成严重影响。治疗新生儿肺炎的方法有很多种，具体治疗方案需要根据孩子的具体情况和医生的建议来确定。一般来说，治疗新生儿肺炎的方法包括药物治疗、物理治疗和综合治疗等。药物治疗是治疗新生儿肺炎的主要方法，医生会根据孩子的年龄和身体状况，选择合适的药物，给予适当的剂量，及时治疗肺炎。物理治疗包括按摩、热敷、吸氧等，可以缓解孩子的疼痛和不适，促进康复。综合治疗是指综合运用药物治疗、物理治疗和其他治疗方法，全面治疗新生儿肺炎，提高治疗效果，减少并发症的发生。希望宝宝能够及时得到治疗，早日康复，健康成长！祝愿你的宝宝早日摆脱肺炎的困扰，健康成长！

PPO-110:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎是一种常见的疾病，需要及时治疗，否则会对孩子的健康造成严重影响。针对新生儿肺炎的治疗，建议及时就医，寻求专业医生的指导和帮助。他们会根据孩子的具体情况，给出最合适的治疗方案，帮助宝宝早日康复，健康成长。希望宝宝能早日摆脱肺炎的困扰，茁壮成长！

PPO-125:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎是一种常见的疾病，需要及时治疗，确保宝宝能够健康成长。建议你咨询专业医生，他们会根据宝宝的情况给出最合适的治疗方案。希望你的宝宝早日康复，远离肺炎的困扰！

PPO-150:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎是一种常见的疾病，需要及时治疗。建议及时就医，寻求专业医生的指导和治疗。医生会根据宝宝的情况，给出最合适的治疗方案。希望小可爱能早日康复，摆脱肺炎的困扰，健康成长！

PPO-200:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？ 新生儿肺炎是一种常见的疾病，需要及时治疗。希望你能早日康复，摆脱肺炎的困扰！记得及时就医，听从专业医生的指导和治疗哦！

PPO-250:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？希望你能早日康复，远离肺炎的困扰！

PPO-300:
我家宝宝出生没几天就有点肺炎了我们赶紧带孩子去检查医生确诊了是肺炎请问我们应该怎么治疗宝宝才能好的利索翱请问新生儿肺炎怎么治疗？希望你能早日康复，远离肺炎的困扰！记得及时就医，寻求专业医生的指导和治疗哦！
```
