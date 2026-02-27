---
title: 20230917 - 测试 codellama 微调 llama-recipes
confluence_page_id: 2589134
created_at: 2023-09-17T15:52:47+00:00
updated_at: 2023-09-19T05:21:08+00:00
---

<https://github.com/facebookresearch/llama-recipes>

# 训练命令

```
CUDA_VISIBLE_DEVICES=0 python -m llama_recipes.finetuning  --use_peft --peft_method lora --model_name codellama/CodeLlama-7b-hf --output_dir /home/user01/sql_audit/codellama-finetune/test-model --use_fp16 --quantization --batch_size_training=1
``` 

# inference.py改写

```
# Copyright (c) Meta Platforms, Inc. and affiliates.
# This software may be used and distributed according to the terms of the Llama 2 Community License Agreement.

# from accelerate import init_empty_weights, load_checkpoint_and_dispatch

import fire
import os
import sys
import time

import torch
from transformers import LlamaTokenizer

from llama_recipes.inference.safety_utils import get_safety_checker
from llama_recipes.inference.model_utils import load_model, load_peft_model

def main(
    model_name,
    peft_model: str=None,
    quantization: bool=False,
    max_new_tokens =100, #The maximum numbers of tokens to generate
    prompt_file: str=None,
    seed: int=42, #seed value for reproducibility
    do_sample: bool=True, #Whether or not to use sampling ; use greedy decoding otherwise.
    min_length: int=None, #The minimum length of the sequence to be generated, input prompt + min_new_tokens
    use_cache: bool=True,  #[optional] Whether or not the model should use the past last key/values attentions Whether or not the model should use the past last key/values attentions (if applicable to the model) to speed up decoding.
    top_p: float=1.0, # [optional] If set to float < 1, only the smallest set of most probable tokens with probabilities that add up to top_p or higher are kept for generation.
    temperature: float=1.0, # [optional] The value used to modulate the next token probabilities.
    top_k: int=50, # [optional] The number of highest probability vocabulary tokens to keep for top-k-filtering.
    repetition_penalty: float=1.0, #The parameter for repetition penalty. 1.0 means no penalty.
    length_penalty: int=1, #[optional] Exponential penalty to the length that is used with beam-based generation. 
    enable_azure_content_safety: bool=False, # Enable safety check with Azure content safety api
    enable_sensitive_topics: bool=False, # Enable check for sensitive topics using AuditNLG APIs
    enable_salesforce_content_safety: bool=True, # Enable safety check with Salesforce safety flan t5
    max_padding_length: int=None, # the max padding length to be used with tokenizer padding the prompts.
    use_fast_kernels: bool = False, # Enable using SDPA from PyTroch Accelerated Transformers, make use Flash Attention and Xformer memory-efficient kernels
    **kwargs
):
    if prompt_file is not None:
        assert os.path.exists(
            prompt_file
        ), f"Provided Prompt file does not exist {prompt_file}"
        with open(prompt_file, "r") as f:
            user_prompt = "\n".join(f.readlines())
    elif not sys.stdin.isatty():
        user_prompt = "\n".join(sys.stdin.readlines())
    else:
        print("No user prompt provided. Exiting.")
        sys.exit(1)
    
    # Set the seeds for reproducibility
    torch.cuda.manual_seed(seed)
    torch.manual_seed(seed)
    
    model = load_model(model_name, quantization)
    if peft_model:
        model = load_peft_model(model, peft_model)
        model = model.to('cuda')

    model.eval()
    
    if use_fast_kernels:
        """
        Setting 'use_fast_kernels' will enable
        using of Flash Attention or Xformer memory-efficient kernels 
        based on the hardware being used. This would speed up inference when used for batched inputs.
        """
        try:
            from optimum.bettertransformer import BetterTransformer
            model = BetterTransformer.transform(model)    
        except ImportError:
            print("Module 'optimum' not found. Please install 'optimum' it before proceeding.")

    tokenizer = LlamaTokenizer.from_pretrained(model_name)
    tokenizer.pad_token = tokenizer.eos_token
    
#    safety_checker = get_safety_checker(enable_azure_content_safety,
#                                        enable_sensitive_topics,
#                                        enable_salesforce_content_safety,
#                                        )
#
#    # Safety check of the user prompt
#    safety_results = [check(user_prompt) for check in safety_checker]
#    are_safe = all([r[1] for r in safety_results])
#    if are_safe:
#        print("User prompt deemed safe.")
#        print(f"User prompt:\n{user_prompt}")
#    else:
#        print("User prompt deemed unsafe.")
#        for method, is_safe, report in safety_results:
#            if not is_safe:
#                print(method)
#                print(report)
#        print("Skipping the inference as the prompt is not safe.")
#        sys.exit(1)  # Exit the program with an error status
        
    batch = tokenizer(user_prompt, padding='max_length', truncation=True, max_length=max_padding_length, return_tensors="pt")

    batch = {k: v.to("cuda") for k, v in batch.items()}
    start = time.perf_counter()
    with torch.no_grad():
        outputs = model.generate(
            **batch,
            max_new_tokens=max_new_tokens,
            do_sample=do_sample,
            top_p=top_p,
            temperature=temperature,
            min_length=min_length,
            use_cache=use_cache,
            top_k=top_k,
            repetition_penalty=repetition_penalty,
            length_penalty=length_penalty,
            **kwargs 
        )
    e2e_inference_time = (time.perf_counter()-start)*1000
    print(f"the inference time is {e2e_inference_time} ms")
    output_text = tokenizer.decode(outputs[0], skip_special_tokens=True)
    
#    # Safety check of the model output
#    safety_results = [check(output_text) for check in safety_checker]
#    are_safe = all([r[1] for r in safety_results])
#    if are_safe:
#        print("User input and model output deemed safe.")
#        print(f"Model output:\n{output_text}")
#    else:
#        print("Model output deemed unsafe.")
#        for method, is_safe, report in safety_results:
#            if not is_safe:
#                print(method)
#                print(report)
    print(f"Model output:\n{output_text}")

if __name__ == "__main__":
    fire.Fire(main)
``` 

# 初始测试: 

数据集是默认的samsum

<https://huggingface.co/datasets/samsum/viewer/samsum/train?row=1>

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/codellama-finetune# cat prompt.txt | python ./inference.py --model_name codellama/CodeLlama-7b-hf --quantization true
Loading checkpoint shards: 100%|█████████████████████████████████████████████████████████████████████████████████| 2/2 [00:03<00:00,  1.53s/it]
The tokenizer class you load from this checkpoint is not the same type as the class this function is called from. It may result in unexpected tokenization.
The tokenizer class you load from this checkpoint is 'CodeLlamaTokenizer'.
The class this function is called from is 'LlamaTokenizer'.
You are using the default legacy behaviour of the <class 'transformers.models.llama.tokenization_llama.LlamaTokenizer'>. If you see this, DO NOT PANIC! This is expected, and simply means that the `legacy` (previous) behavior will be used so nothing changes for you. If you want to use the new behaviour, set `legacy=False`. This should only be set if you understand what it means, and thouroughly read the reason why this was added as explained in https://github.com/huggingface/transformers/pull/24565
Asking to pad to max_length but no maximum length is provided and the model has no predefined maximum length. Default to no padding.
Asking to truncate to max_length but no maximum length is provided and the model has no predefined maximum length. Default to no truncation.
Setting `pad_token_id` to `eos_token_id`:2 for open-end generation.
the inference time is 16458.272974938154 ms
Model output:
Summarize this dialog:

Olivia: Who are you voting for in this election? Oliver: Liberals as always. Olivia: Me too!! Oliver: Great

---

Summary:
> Olivia: Who are you voting for in this election? Oliver: Liberals as always. Olivia: Me too!! Oliver: Great

> Opposite:
>
> Opposite: Who are you voting for in this election?

#### 4)

![](https://s3-us-west-2.amazonaws.com/appacitive/assets/data-enrichment/empathy_for_other
``` 

#   
微调方法

数据集是默认的samsum

```
CUDA_VISIBLE_DEVICES=0 python -m llama_recipes.finetuning  --use_peft --peft_method lora --model_name codellama/CodeLlama-7b-hf --output_dir /home/user01/sql_audit/codellama-finetune/test-model --use_fp16 --quantization --batch_size_training=1
``` 

# 微调效果测试

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/codellama-finetune# cat prompt.txt | python ./inference.py --model_name codellama/CodeLlama-7b-hf --peft_model /home/user01/sql_audit/codellama-finetune/test-model --quantization true
Loading checkpoint shards: 100%|█████████████████████████████████████████████████████████████████████████████████| 2/2 [00:03<00:00,  1.53s/it]
The tokenizer class you load from this checkpoint is not the same type as the class this function is called from. It may result in unexpected tokenization.
The tokenizer class you load from this checkpoint is 'CodeLlamaTokenizer'.
The class this function is called from is 'LlamaTokenizer'.
You are using the default legacy behaviour of the <class 'transformers.models.llama.tokenization_llama.LlamaTokenizer'>. If you see this, DO NOT PANIC! This is expected, and simply means that the `legacy` (previous) behavior will be used so nothing changes for you. If you want to use the new behaviour, set `legacy=False`. This should only be set if you understand what it means, and thouroughly read the reason why this was added as explained in https://github.com/huggingface/transformers/pull/24565
Asking to pad to max_length but no maximum length is provided and the model has no predefined maximum length. Default to no padding.
Asking to truncate to max_length but no maximum length is provided and the model has no predefined maximum length. Default to no truncation.
Setting `pad_token_id` to `eos_token_id`:2 for open-end generation.
the inference time is 2680.1196397282183 ms
Model output:
Summarize this dialog:

Olivia: Who are you voting for in this election? Oliver: Liberals as always. Olivia: Me too!! Oliver: Great

---

Summary:
Olive and Oliver are voting for the Liberal Party.
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/codellama-finetune#
``` 

效果正确

# 进行infilling任务

按以上方式进行infilling任务会失败, 输入: 

```
def sum(a,b:int):
    <FILL_ME>
    return c
``` 

输出: 

```
def sum(a,b):
    <FILL_ME>
    return c

def sum(a,b):
    <FILL_ME>
    return c

def sum(a,b):
    <FILL_ME>
    return c

def sum(a,b):
    <FILL_ME>
    return c

def sum(a,b):
    <FILL_ME
``` 

调整tokenizer为 transformers.models.code_llama.tokenization_code_llama_fast.CodeLlamaTokenizerFast, 效果正确: 

```
c = a+b
``` 

同时, 之前微调的效果仍然保持: 

```
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/codellama-finetune# python ./code_infilling_example.py --model_name codellama/CodeLlama-7b-hf --peft_model /home/user01/sql_audit/codellama-finetune/test-model --quantization true --prompt_file prompt.txt --use_fast_kernels False
Loading checkpoint shards: 100%|███████████████████████████████████████████████████████████████████████| 2/2 [00:02<00:00,  1.47s/it]
<class 'transformers.models.code_llama.tokenization_code_llama_fast.CodeLlamaTokenizerFast'>
Setting `pad_token_id` to `eos_token_id`:2 for open-end generation.
the inference time is 3154.149533249438 ms
Olivia and Oliver are voting for the Liberals in this election.
``` 

# TODO: 测试对代码进行微调

自定义数据集的载入方式: 

```
from datasets import load_dataset
from pathlib import Path

from torch.utils.data import Dataset

from llama_recipes.datasets.utils import ConcatDataset

class csv_dataset(Dataset):
    def __init__(
        self,
        tokenizer,
        csv_name=None,
    ):

        try:
            self.dataset = load_dataset(
                "csv",
                data_files={"train": [csv_name]},  # "eval": "grammar_validation.csv"},
                delimiter=",",
            )
        except Exception as e:
            print("Loading of grammar dataset failed! Please see recipes/ft_datasets/grammar_dataset/grammar_dataset_process.ipynb for details on how to download the dataset.")
            raise e

        # if num_samples:
        #    self.dataset = self.dataset.select(list(range(0, num_samples)))
        
        self.tokenizer = tokenizer
        self.print_text = False  # print_text

    def __len__(self):
        return self.dataset["train"].shape[0]

    def convert_to_features(self, example_batch):
        # Create prompt and tokenize contexts and questions

        if self.print_text:
            print("Input Text: ", self.clean_text(example_batch["text"]))

        #input_ = example_batch["input"]
        #target_ = example_batch["target"]
        #prompt = f"[INST]complete the code[/INST]: \n{input_}\n---\n[CODE]{target_}[/CODE]"
        #sample = self.tokenizer(prompt)
        sample = self.tokenizer(example_batch["prompt"])
        
        return sample

    def __getitem__(self, index):
        sample = self.convert_to_features(self.dataset["train"][index])
        source_ids = sample["input_ids"]
        src_mask = sample["attention_mask"]

        return {
            "input_ids": source_ids,
            "attention_mask": src_mask,
            "labels": source_ids.copy(),
        }

def get_custom_dataset(dataset_config, tokenizer, split):
    dataset = csv_dataset(
        tokenizer=tokenizer,
        csv_name="train.csv",
    )
    
    return ConcatDataset(dataset, chunk_size=70)
``` 

  - 注意以上的chunk_size: 这段代码将csv中的数据进行tokenizer, 然后按照chunk_size进行切割. 如果chunk_size太小, 那么一次训练的上下文太少, 无法达成效果. 如果chunk_size太大, 最后一组不完整的chunk会被丢弃, 导致训练数据丢失

csv数据: 

```
prompt,
"<s>[INST] <<SYS>> Given the purpose and name of a python function, write the function <</SYS>> purpose: concat two numbers. name: add_two_numbers [/INST] [CODE]def add_two_numbers(a,b):
    return str(a) + str(b)[/CODE]</s>",
"<s>[INST] <<SYS>> Given the purpose and name of a python function, write the function <</SYS>> purpose: concat two numbers. name: add_two_numbers [/INST] [CODE]def add_two_numbers(a,b):
    return str(a) + str(b)[/CODE]</s>",
"<s>[INST] <<SYS>> Given the purpose and name of a python function, write the function <</SYS>> purpose: concat two numbers. name: add_two_numbers [/INST] [CODE]def add_two_numbers(a,b):
    return str(a) + str(b)[/CODE]</s>",
"<s>[INST] <<SYS>> Given the purpose and name of a python function, write the function <</SYS>> purpose: concat two numbers. name: add_two_numbers [/INST] [CODE]def add_two_numbers(a,b):
    return str(a) + str(b)[/CODE]</s>",
``` 

微调命令: 

```
CUDA_VISIBLE_DEVICES=0 python -m llama_recipes.finetuning  --use_peft --peft_method lora --model_name codellama/CodeLlama-7b-hf --output_dir /home/user01/sql_audit/codellama-finetune/test-model --use_fp16 --quantization --batch_size_training=1 --dataset custom_dataset --file dataset.py
``` 

效果: 

```
//微调前
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/codellama-finetune# python ./code_infilling_example.py --model_name codellama/CodeLlama-7b-hf --quantization --prompt_file test.prompt.txt --use_fast_kernels False --temperature 0.6
Loading checkpoint shards: 100%|██████████████████████████████████████████████████████| 2/2 [00:02<00:00,  1.46s/it]
<class 'transformers.models.code_llama.tokenization_code_llama_fast.CodeLlamaTokenizerFast'>
Setting `pad_token_id` to `eos_token_id`:2 for open-end generation.
the inference time is 7180.357689037919 ms
[CODE] def add_two_numbers(a, b): return a + b [/CODE] [CODE] print(add_two_numbers(2, 3))
 
 
//微调后
(chatglm_etuning) root@gpu198:/home/user01/sql_audit/codellama-finetune# python ./code_infilling_example.py --model_name codellama/CodeLlama-7b-hf --peft_model /home/user01/sql_audit/codellama-finetune/test-model --quantization --prompt_file prompt2.txt --use_fast_kernels False --temperature 0.6
Loading checkpoint shards: 100%|██████████████████████████████████████████████████████| 2/2 [00:03<00:00,  1.51s/it]
peft_model=/home/user01/sql_audit/codellama-finetune/test-model
<class 'transformers.models.code_llama.tokenization_code_llama_fast.CodeLlamaTokenizerFast'>
Setting `pad_token_id` to `eos_token_id`:2 for open-end generation.
the inference time is 4595.821323338896 ms
def add_two_numbers(a,b):
    return str(a) + str(b)
```
