---
title: 20240914 - 使用 Llama-factory 做强化学习
confluence_page_id: 3342394
created_at: 2024-09-14T05:58:51+00:00
updated_at: 2024-09-25T16:53:18+00:00
---

# 目标

尝试强化学习

# 环境

10.186.16.135, root/action123

eval "$(/data/huangyan/anaconda3/bin/conda shell.bash hook)"

conda create --name huangyan-ppo python=3.9

conda activate huangyan-ppo

启动jupyter的命令: 

```
nohup jupyter lab --allow-root --ip=0.0.0.0 --no-browser &
``` 

# 安装Llama-factory

```
git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
ALL_PROXY=http://10.186.16.136:7890 pip install -e ".[torch,metrics]"
``` 

启动GUI: 

```
llamafactory-cli webui
``` 

通过GUI可获得命令: 

```
llamafactory-cli train \
    --stage ppo \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset belle_0.5m \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 3.0 \
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
    --output_dir saves/Qwen2-7B/lora/train_2024-09-14-08-20-31 \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --reward_model saves/Qwen2-7B/lora/rewards2 \
    --reward_model_type lora \
    --top_k 0 \
    --top_p 0.9 
``` 

### 问题1: 数据集加载很慢

手工测试

```
HTTPS_PROXY=http://10.186.16.136:7890 python -c "import logging; logging.basicConfig(level=logging.DEBUG); from datasets import load_dataset; dataset = load_dataset('BelleGroup/train_0.5M_CN'); print(dataset)"
``` 

![image2024-9-14 16:37:34.png](/assets/01KJBZFC97Q7JPHD2GFX6VGG92/image2024-9-14%2016%3A37%3A34.png)

代理没问题, 但加载确实很慢

手工找到模型路径, 准备手工下载: 

```
ALL_PROXY=http://10.186.16.136:7890 python -c "from datasets import load_dataset_builder; builder = load_dataset_builder('BelleGroup/train_0.5M_CN'); print(builder.cache_dir)"
 
输出: 
/root/.cache/huggingface/datasets/BelleGroup___train_0.5_m_cn/default/0.0.0/29c35e9b56c7fc91f04a613ed33896f7a19bad54
``` 

# 训练Reward Model

问题: 目前使用了 偏好数据集 来训练RM, 不知道是否合适

偏好数据集选择了 flammenai/Date-DPO-v3 (这个数据集 主要为了修正 GPT-speak), 在dataset_info.json中增加定义: 

```
  "date-dpo-v3": {
    "hf_hub_url": "flammenai/Date-DPO-v3",
    "ms_hub_url": "flammenai/Date-DPO-v3",
    "ranking": true,
    "columns": {
      "prompt": "prompt",
      "chosen": "chosen",
      "rejected": "rejected"
    }
  },
``` 

命令: 

```
PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=1 ALL_PROXY=http://10.186.16.136:7890 llamafactory-cli train \
    --stage rm \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset orca_pairs \
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
    --output_dir saves/Qwen2-7B/lora/train_2024-09-15-12-37-28 \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all
``` 

# 训练PPO

增加数据集定义: 

```
 "roleplay_set": {
    "hf_hub_url": "Rithik002/roleplay_set",
    "ms_hub_url": "Rithik002/roleplay_set",
    "columns": {
      "prompt": "input",
      "response": "output",
      "system": "instruction"
    }
  }
``` 

训练命令:

```
PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=1 ALL_PROXY=http://10.186.16.136:7890 llamafactory-cli train \
    --stage ppo \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset roleplay_set \
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
    --output_dir saves/Qwen2-7B/lora/train_2024-09-15-17-10-26 \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --reward_model saves/Qwen2-7B/lora/train_2024-09-15-12-37-28 \
    --reward_model_type lora \
    --top_k 0 \
    --top_p 0.9
``` 

从数据集中抽取的测试数据, 效果并不好: 

![image2024-9-18 11:6:33.png](/assets/01KJBZFC97Q7JPHD2GFX6VGG92/image2024-9-18%2011%3A6%3A33.png)

# 先微调?

选取数据集: 

PPO reward数据集: Undi95/Weyaxi-humanish-dpo-project-noemoji (更像人类的回答)

sft, PPO 数据集: Lines/Open-Domain-Oral-Disease-QA-Dataset (医疗问答数据集)

训练reward:

```
PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 llamafactory-cli train \
    --stage rm \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset Weyaxi-humanish-dpo-project-noemoji \
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
    --output_dir saves/Qwen2-7B/lora/reward\
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all
``` 

loss快速下降到0, 可能是学习率过高, 调整成5e-6

![image2024-9-18 17:28:54.png](/assets/01KJBZFC97Q7JPHD2GFX6VGG92/image2024-9-18%2017%3A28%3A54.png)

在200例之后, loss归零.

  1. 尝试理解reward模型的输入和输出
  2. 使用100/200/600的模型, 测试情况

# 对reward模型的输入/输出的理解

输入: 

  - 从run_rm函数进行分析: 
    - 数据集, 经过preprocess_pairwise_dataset的处理 (代码: get_preprocess_and_print_func)
    - 在数据聚合时, 使用了 PairwiseDataCollatorWithPadding, 其中逻辑是将 前一半放正例, 后一半放反例: 

```
@dataclass
class PairwiseDataCollatorWithPadding(MultiModalDataCollatorForSeq2Seq):
    r"""
    Data collator for pairwise data.
    """

    def __call__(self, features: Sequence[Dict[str, Any]]) -> Dict[str, "torch.Tensor"]:
        r"""
        Pads batched data to the longest sequence in the batch.

        We generate 2 * n examples where the first n examples represent chosen examples and
        the last n examples represent rejected examples.
        """
        concatenated_features = []
        for key in ("chosen", "rejected"):
            for feature in features:
                target_feature = {
                    "input_ids": feature["{}_input_ids".format(key)],
                    "attention_mask": feature["{}_attention_mask".format(key)],
                    "labels": feature["{}_labels".format(key)],
                    "images": feature["images"],
                    "videos": feature["videos"],
                }
                concatenated_features.append(target_feature)

        return super().__call__(concatenated_features)
```

    - 计算loss时: (PairwiseTrainer.compute_loss)
      - 模型采集 前一半用例的输出的最后一个有效字符的"分数" 作为 正例的评分
      - 模型采集 后一半用例的输出的最后一个有效字符的"分数" 作为 反例的评分
      - loss = logsigmoid(正例评分 - 反例评分), 将评分拉开
      - 为什么用最后一个有效字符的分数代表整体的分数: 
        - "在现代的深度学习模型（特别是Transformer架构）中，每个词的表示都依赖于整个序列的上下文。当模型处理 "Hello world" 时，"world" 的最终表示已经包含了 "Hello" 的信息。"

用代码进行model generate测试: [TestRewardModel.html](/assets/01KJBZFC97Q7JPHD2GFX6VGG92/TestRewardModel.html)

在使用reward model时, 不是使用generate模式, 而仅使用一次调用

(一次调用预测下一个token, generate反复调用不断生成后续token)

一次调用, 生成下一个token时, 会生成 vocabs (一个词汇表, 以及每个词在下一个token位置出现的概率), 经过换算形成一个评分A.

这个换算的过程, 由 AutoModelForCausalLMWithValueHead进行.

AutoModelForCausalLMWithValueHead的原理? reward model训练的主要目标是 "价值头", 目前好像丢失了??

# 分析 AutoModelForCausalLMWithValueHead

<https://github.com/huggingface/trl/blob/663002f609ff8af812620df88b45341cc5056786/trl/models/modeling_value_head.py#L140>

AutoModelForCausalLMWithValueHead.forward 中: 将基模型的最后一个hidden_states, 通过v_head层映射成一个数值:

```
        base_model_output = self.pretrained_model(
            input_ids=input_ids,
            attention_mask=attention_mask,
            **kwargs,
        )

        last_hidden_state = base_model_output.hidden_states[-1]
        lm_logits = base_model_output.logits
        loss = base_model_output.loss

        if last_hidden_state.device != self.v_head.summary.weight.device:
            last_hidden_state = last_hidden_state.to(self.v_head.summary.weight.device)

        value = self.v_head(last_hidden_state).squeeze(-1)

        # force upcast in fp32 if logits are in half-precision
        if lm_logits.dtype != torch.float32:
            lm_logits = lm_logits.float()

        if return_past_key_values:
            return (lm_logits, loss, value, base_model_output.past_key_values)
        else:
            return (lm_logits, loss, value)
``` 

v_head层: 使用了一个dropout层和一个Linear层, 作为一个映射组件

```
class ValueHead(nn.Module):
    r"""
    The ValueHead class implements a head for GPT2 that returns a scalar for each output token.
    """

    def __init__(self, config, **kwargs):
		...
        self.dropout = nn.Dropout(summary_dropout_prob) if summary_dropout_prob else nn.Identity()
		...
        self.summary = nn.Linear(hidden_size, 1)
		...
        self.flatten = nn.Flatten()

    def forward(self, hidden_states):
        output = self.dropout(hidden_states)

        # For now force upcast in fp32 if needed. Let's keep the
        # output in fp32 for numerical stability.
        if output.dtype != self.summary.weight.dtype:
            output = output.to(self.summary.weight.dtype)

        output = self.summary(output)
        return output
``` 

v_head的状态如何保存: 放在基模型的状态中, 以v_head.开头

```
    def state_dict(self, *args, **kwargs):
...
        v_head_state_dict = self.v_head.state_dict(*args, **kwargs)
        for k, v in v_head_state_dict.items():
            pretrained_model_state_dict[f"v_head.{k}"] = v
``` 

# 评估Reward Model

代码: [TestRewardModel (1).html](/assets/01KJBZFC97Q7JPHD2GFX6VGG92/TestRewardModel%20%281%29.html)

重点:

  - 模型需要用AutoModelForCausalLMWithValueHead进行包装
  - 依次读取: 预训练模型, 模型的微调层, v_head层
  - 模型切换到eval模式

用测试数据集进行eval, 评估微调的结果, 更新代码: [TestRewardModel (2).html](/assets/01KJBZFC97Q7JPHD2GFX6VGG92/TestRewardModel%20%282%29.html)

其中发现多次载入模型, 其评分会不一样, 需要固定随机数生成器:

```
import torch
import numpy as np
import random

def set_seed(seed):
    torch.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)
    np.random.seed(seed)
    random.seed(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False

# 使用方法
set_seed(42)  # 训练的默认参数中, seed=42
``` 

## 准确率不准的问题

用200个训练用例, 进行准确率评估: 

  - 如之前的微调参数, epoch = 3.0, lr = 5e-5, cutoff_len = 1024 -> loss = 0, 准确率是 76%
  - epoch = 4.0, lr = 5e-5, cutoff_len = 1024 -> loss = 0, 准确率是 80%
  - epoch = 3.0, lr = 1e-5, cutoff_len = 1024 -> loss = 0, 准确率是 76%
  - epoch = 3.0, lr = 1e-5, 修改loss函数 增加了margin, cutoff_len = 1024 -> loss = 0, 准确率是 76%
    - margin相当于设置了 正例 - 反例 >= margin 这个门槛, 让正例和反例之间的差别更大

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

        margin = 0.1
        loss = -torch.nn.functional.logsigmoid(chosen_scores.float() - rejected_scores.float()).mean()
        print(f"1-loss={loss}")
        loss = -torch.nn.functional.logsigmoid(chosen_scores.float() - rejected_scores.float() + margin).mean()
        print(f"2-loss={loss}")
        if return_outputs:
            return loss, (loss, chosen_scores, rejected_scores)
        else:
            return loss

``` 

  - epoch = 3.0, lr = 1e-5, cutoff_len = 2048 -> loss = 0, 准确率是 77%

做不出明显差异.

怀疑是验证代码有问题? 

使用微调的eval模式进行评估: 

```
PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=0 ALL_PROXY=http://10.186.16.136:7890 llamafactory-cli train \
    --stage rm \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template default \
    --flash_attn auto \
    --dataset_dir data \
    --dataset Weyaxi-humanish-dpo-project-noemoji \
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
    --output_dir saves/Qwen2-7B/lora/reward7\
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
    --eval_dataset Weyaxi-humanish-dpo-project-noemoji \
    --per_device_eval_batch_size 1 \
    --eval_steps 20 

``` 

发现仅在20 step后, eval的结果就很好, 准确率已经达到1.0 : 

```
{'eval_loss': 0.0026337241288274527, 'eval_accuracy': 1.0, 'eval_runtime': 155.3821, 'eval_samples_per_second': 9.834, 'eval_steps_per_second': 9.834, 'epoch': 0.1, 'num_input_tokens_seen': 62432}
``` 

LlamaFactory的准确率计算: 

```
@dataclass
class ComputeAccuracy:
...

    def __call__(self, eval_preds: "EvalPrediction", compute_result: bool = True) -> Optional[Dict[str, float]]:
        chosen_scores, rejected_scores = numpify(eval_preds.predictions[0]), numpify(eval_preds.predictions[1])
        if not chosen_scores.shape:
            self.score_dict["accuracy"].append(chosen_scores > rejected_scores)
        else:
            for i in range(len(chosen_scores)):
                self.score_dict["accuracy"].append(chosen_scores[i] > rejected_scores[i])

        if compute_result:
            return self._dump()
``` 

准确率的定义跟手工脚本一致, 认为手工脚本有问题. 尝试: 

  1. 怀疑模型初始化有问题, 在脚本中直接使用LlamaFactory的初始化方法加载模型, 准确率仍然有问题
  2. 怀疑数据的formatter有问题, 在_get_preprocessed_dataset打印了数据样例: 
     1. ![image2024-9-25 14:49:32.png](/assets/01KJBZFC97Q7JPHD2GFX6VGG92/image2024-9-25%2014%3A49%3A32.png)
     2. 发现数据结尾是"<|endoftext|>"
     3. 将数据结尾增加后, 准确率已经正常

修订后的脚本: [TestRewardModel.ipynb](/assets/01KJBZFC97Q7JPHD2GFX6VGG92/TestRewardModel.ipynb) , 脚本中的重点: 

  1. 可使用LlamaFactory进行模型加载, 也可使用原生方式
  2. 原生加载时: 
     1. 加载原始模型
     2. 生成一个peft-lora 空层 (这一步没必要, 是因为LlamaFactory认为 这个基模型准备进行微调, 同时要复用另一个微调层作为 Reward Model)
     3. 加载RM的微调层
     4. 加载RM的vhead层

## 问题: 在少量训练后, 准确率就已经100%

符合loss迅速减到0的现象, 其过拟合过快, 泛化能力目前未知

判断是因为数据集选错了. 

想找 验证过可以做PPO的数据集, 来跑通当前流程.

# 额外参考

  - 使用reward函数, 而不是reward model的PPO样例: <https://github.com/arunprsh/ChatGPT-Decoded-GPT2-FAQ-Bot-RLHF-PPO/blob/main/04-ppo/01-rlhf.ipynb>
  - 这个脚本中, 用ref_model和model比较, 来评估PPO前后的效果: <https://github.com/huggingface/trl/blob/main/examples/notebooks/gpt2-sentiment.ipynb>
