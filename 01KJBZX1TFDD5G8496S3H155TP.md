---
title: 20250810 - 使用nsys, 分析A100 8卡的时间消耗
confluence_page_id: 4161946
created_at: 2025-08-10T07:07:01+00:00
updated_at: 2025-08-10T16:37:31+00:00
---

在 Mac 下安装NVIDIA Nsight Systems

使用ssh方式, 连到远程机器上, 命令行通过ms-swift的日志获取详细的命令: 

```
/root/anaconda3/condabin/conda run -n huangyan python -m torch.distributed.run --nproc_per_node 8 /data/huangyan/ms-swift/swift/cli/rlhf.py --rlhf_type grpo --train_type lora --torch_dtype bfloat16 --per_device_train_batch_size 1 --per_device_eval_batch_size 1 --learning_rate 1e-6 --lora_rank 8 --lora_alpha 32 --gradient_accumulation_steps 1 --eval_steps 10 --save_steps 10 --save_total_limit 30 --logging_steps 5 --warmup_ratio 0.05 --gc_collect_after_offload true --max_length 6400 --max_completion_length 6400 --num_generations 2 --log_completions true --log_entropy true --ddp_find_unused_parameters false --use_vllm true --output_dir /data/huangyan/sql-ai-2/train_logs/2025-08-10_01-46-52/train_output --vllm_max_model_len 6400 --vllm_gpu_memory_utilization 0.60 --offload_optimizer true --offload_model true --sleep_level 1 --loss_type bnpo --epsilon_high 0.28 --dynamic_sample true --max_resample_times 6 --overlong_filter true --soft_cache_length 4096 --soft_max_length 8096 --model /root/.cache/modelscope/hub/models/Qwen/Qwen3-8B --dataset /data/huangyan/sql-ai-2/train_logs/2025-08-10_01-46-52/train_data.grpo.jsonl --val_dataset /data/huangyan/sql-ai-2/train_logs/2025-08-10_01-46-52/val_data.grpo.jsonl --external_plugins /data/huangyan/sql-ai-2/grpo/reward_function.py --reward_funcs sql_ai_action3_f1 sql_ai_completeness sql_ai_action_count sql_ai_format_error --reward_weights 1.0 1.0 1.0 1.0 --num_generations 8 --per_device_train_batch_size 2 --per_device_eval_batch_size 1 --gradient_accumulation_steps 4 --attn_impl flash_attn --dataloader_num_workers 4 --learning_rate 1e-4 --num_iterations 4 --max_length 16000 --max_completion_length 16000 --vllm_max_model_len 16000 --importance_sampling_level sequence --log_entropy true --top_entropy_quantile 0.2 --num_train_epochs 20 --resume_only_model --ignore_data_skip --resume_from_checkpoint /data/huangyan/sql-ai-2/train_logs/2025-08-09_13-26-26/train_output/v0-20250809-132637/checkpoint-84
``` 

![image2025-8-10 15:4:56.png](/assets/01KJBZX1TFDD5G8496S3H155TP/image2025-8-10%2015%3A4%3A56.png)

得到报告: 

![image2025-8-10 15:6:14.png](/assets/01KJBZX1TFDD5G8496S3H155TP/image2025-8-10%2015%3A6%3A14.png)

注意: 如果使用nsys获得报告, 再传到MAC上读取, 会没有CUDA HW行, 应该是nsys的命令行缺少参数, 目前先不研究

默认项目目录: /Users/tachikoma/.nsightsystems/Projects/Project\ 4/report1.nsys-rep

蓝色的是CUDA kernel running, 橙色是Graph running

![image2025-8-11 0:37:3.png](/assets/01KJBZX1TFDD5G8496S3H155TP/image2025-8-11%200%3A37%3A3.png)
