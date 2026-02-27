---
title: 20250811 - 诊断GRPO训练过程的时间/内存消耗
confluence_page_id: 4161951
created_at: 2025-08-11T04:56:05+00:00
updated_at: 2025-08-13T09:47:03+00:00
---

# SwanLab

在ms-swift启动参数中, 增加参数: 

```
--report_to swanlab \
--swanlab_project sql-ai-2
``` 

ms-swift中, 使用 patch_profiling_context 和 patch_profiling_decorator 来标记被性能监控的代码块, 其中包括: 

  - generate (_engine_infer)
  - 每个reward function
  - _prepare_inputs (包括 _generate_and_score_completions)
  - _move_model_to_vllm
  - compute_loss
  - _get_per_token_logps_and_entropies
  - _get_last_hidden_state

主要集中于GRPO过程中, 由ms-swift中操作的部分 (completion和loss计算), 但是否包括前向后向部分?

另外, 在patch_profiling_context中, 增加 标准输出, 这样可对于调用次数等做出后续分析

在ms-swift启动时, 统一所有日志的输出格式: 

```
import logging
import sys
import os

def print_all_loggers_and_handlers():
    """
    打印出当前所有已知的 logger 及其关联的 handler、level 和 formatter。
    这对于调试复杂的日志配置非常有用。
    """
    print("="*80)
    print("Inspecting all loggers and their handlers...")
    print("="*80)
    
    # 获取根 logger
    root_logger = logging.getLogger()
    
    # 将根 logger 和所有其他 logger 放在一个列表中
    loggers = [root_logger] + [logging.getLogger(name) for name in logging.Logger.manager.loggerDict]
    
    for logger in loggers:
        # 只显示有 handler 的 logger，或者根 logger
        if logger.hasHandlers() or logger.name == 'root':
            print(f"\n--- Logger: '{logger.name}' (Level: {logging.getLevelName(logger.level)}, Propagate: {logger.propagate}) ---")
            
            if not logger.hasHandlers():
                print("    No handlers directly attached.")
            else:
                for handler in logger.handlers:
                    print(f"  -> Handler: {handler.__class__.__name__} (Level: {logging.getLevelName(handler.level)})")
                    if handler.formatter:
                        # Formatter 的格式字符串通常存储在 _fmt 属性中
                        formatter_format = getattr(handler.formatter, '_fmt', 'N/A')
                        print(f"     - Formatter: {handler.formatter.__class__.__name__}")
                        print(f"     - Format:    '{formatter_format}'")
                    else:
                        print("     - No formatter set.")
    
    print("\n" + "="*80)
    print("Inspection complete.")
    print("="*80)

def unify_handler_formats(format_string, date_format=None):
    """
    遍历所有已知的 handler，如果 handler 已有 formatter，
    则将其替换为一个新的、统一格式的 formatter。

    :param format_string: 希望统一成的日志格式字符串。
    :param date_format: 希望统一成的日期格式字符串 (可选)。
    """
    print(f"\n>>> Attempting to unify all handler formats to:\n    '{format_string}'\n")
    
    # 1. 创建新的、统一的 Formatter 对象
    new_formatter = logging.Formatter(format_string, datefmt=date_format)
    
    # 2. 获取所有 logger (包括 root)
    root_logger = logging.getLogger()
    all_loggers = [root_logger] + [logging.getLogger(name) for name in logging.Logger.manager.loggerDict]
    
    modified_handlers_count = 0
    
    # 3. 遍历所有 logger
    for logger in all_loggers:
        # 4. 遍历该 logger 的所有 handler
        for handler in logger.handlers:
            # 5. 检查 handler 是否已经有一个 formatter
            if handler.formatter:
                # 如果有，就用新的 formatter 替换它
                handler.setFormatter(new_formatter)
                modified_handlers_count += 1
                print(f"  - Updated formatter for handler {handler.__class__.__name__} on logger '{logger.name}'")

    if modified_handlers_count == 0:
        print("  - No handlers with existing formatters were found to update.")
    else:
        print(f"\nSuccessfully updated {modified_handlers_count} handler(s).")
 
 
...
 

    def run(self):
        ...

        trainer_cls = TrainerFactory.get_trainer_cls(args)

        print_all_loggers_and_handlers()
        UNIFIED_FORMAT = '%(asctime)s %(filename)s:%(lineno)s %(levelname)s:%(name)s p:%(process)d t:%(thread)d: %(message)s'
        UNIFIED_DATE_FORMAT = '%Y-%m-%d %H:%M:%S'
        unify_handler_formats(UNIFIED_FORMAT, UNIFIED_DATE_FORMAT)
        print_all_loggers_and_handlers()

        trainer = trainer_cls(
...
        )
        return self.train(trainer)

``` 

# 对日志进行分析

发现日志中, 在VLLM中, 有一些时间是没有负载记录的

![image2025-8-11 13:57:28.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-11%2013%3A57%3A28.png)

增加调试级别: 

```
TRANSFORMERS_VERBOSITY=detail VLLM_LOGGING_LEVEL=DEBUG NCCL_DEBUG=TRACE CUDA_LAUNCH_BLOCKING=1 
``` 

日志: [grpo_2025-08-11_16-09-30.log.zip](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/grpo_2025-08-11_16-09-30.log.zip)

日志举例: 

```
第一轮生成-加载weight
2025-08-11 16:11:32 utils.py:183 DEBUG:vllm.model_executor.models.utils p:475097 t:140589927618368: Loaded weight lm_head.weight with shape torch.Size([151936, 4096])
2025-08-11 16:11:32 utils.py:183 DEBUG:vllm.model_executor.models.utils p:475098 t:140625347454784: Loaded weight lm_head.weight with shape torch.Size([151936, 4096])
2025-08-11 16:11:32 utils.py:183 DEBUG:vllm.model_executor.models.utils p:475093 t:140305750705984: Loaded weight lm_head.weight with shape torch.Size([151936, 4096])
2025-08-11 16:11:32 utils.py:183 DEBUG:vllm.model_executor.models.utils p:475094 t:140630171514688: Loaded weight lm_head.weight with shape torch.Size([151936, 4096])
2025-08-11 16:11:32 utils.py:183 DEBUG:vllm.model_executor.models.utils p:475096 t:140390354827072: Loaded weight lm_head.weight with shape torch.Size([151936, 4096])

第一轮生成-重置prefix cache
2025-08-11 16:11:32 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475097 t:140589927618368: Successfully reset prefix cache
2025-08-11 16:11:32 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475098 t:140625347454784: Successfully reset prefix cache
2025-08-11 16:11:32 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475093 t:140305750705984: Successfully reset prefix cache
2025-08-11 16:11:32 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475094 t:140630171514688: Successfully reset prefix cache
2025-08-11 16:11:32 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475096 t:140390354827072: Successfully reset prefix cache
2025-08-11 16:11:32 utils.py:183 DEBUG:vllm.model_executor.models.utils p:475095 t:140373120079680: Loaded weight lm_head.weight with shape torch.Size([151936, 4096])
2025-08-11 16:11:32 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475095 t:140373120079680: Successfully reset prefix cache
2025-08-11 16:11:34 executor_base.py:203 INFO:vllm.executor.executor_base p:475098 t:140625347454784: It took 0.260924 seconds to wake up tags ['kv_cache'].
2025-08-11 16:11:34 executor_base.py:203 INFO:vllm.executor.executor_base p:475094 t:140630171514688: It took 0.259688 seconds to wake up tags ['kv_cache'].

第一轮生成-重置kv_cache
2025-08-11 16:11:34 executor_base.py:203 INFO:vllm.executor.executor_base p:475097 t:140589927618368: It took 0.265484 seconds to wake up tags ['kv_cache'].
2025-08-11 16:11:34 executor_base.py:203 INFO:vllm.executor.executor_base p:475093 t:140305750705984: It took 0.269847 seconds to wake up tags ['kv_cache'].
2025-08-11 16:11:34 executor_base.py:203 INFO:vllm.executor.executor_base p:475095 t:140373120079680: It took 0.265793 seconds to wake up tags ['kv_cache'].
2025-08-11 16:11:34 executor_base.py:203 INFO:vllm.executor.executor_base p:475096 t:140390354827072: It took 0.262632 seconds to wake up tags ['kv_cache'].
2025-08-11 16:11:34 executor_base.py:203 INFO:vllm.executor.executor_base p:475092 t:140079487534912: It took 0.287821 seconds to wake up tags ['kv_cache'].
2025-08-11 16:11:35 executor_base.py:203 INFO:vllm.executor.executor_base p:475091 t:139650923292480: It took 2.606242 seconds to wake up tags ['weights'].
2025-08-11 16:11:36 utils.py:183 DEBUG:vllm.model_executor.models.utils p:475091 t:139650923292480: Loaded weight lm_head.weight with shape torch.Size([151936, 4096])
2025-08-11 16:11:36 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475091 t:139650923292480: Successfully reset prefix cache
2025-08-11 16:11:40 executor_base.py:203 INFO:vllm.executor.executor_base p:475091 t:139650923292480: It took 0.455951 seconds to wake up tags ['kv_cache'].
2025-08-11 16:11:59 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475095 t:140373120079680: Successfully reset prefix cache
2025-08-11 16:12:02 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475095 t:140373120079680: Sleep mode freed 71.25 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:12:02 executor_base.py:187 INFO:vllm.executor.executor_base p:475095 t:140373120079680: It took 2.756636 seconds to fall asleep.
2025-08-11 16:12:06 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475096 t:140390354827072: Successfully reset prefix cache
2025-08-11 16:12:07 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475094 t:140630171514688: Successfully reset prefix cache
2025-08-11 16:12:07 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475096 t:140390354827072: Sleep mode freed 71.25 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:12:07 executor_base.py:187 INFO:vllm.executor.executor_base p:475096 t:140390354827072: It took 1.497159 seconds to fall asleep.
2025-08-11 16:12:09 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475093 t:140305750705984: Successfully reset prefix cache
2025-08-11 16:12:11 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475094 t:140630171514688: Sleep mode freed 71.25 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:12:11 executor_base.py:187 INFO:vllm.executor.executor_base p:475094 t:140630171514688: It took 3.793068 seconds to fall asleep.
2025-08-11 16:12:12 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475093 t:140305750705984: Sleep mode freed 71.31 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:12:12 executor_base.py:187 INFO:vllm.executor.executor_base p:475093 t:140305750705984: It took 3.304507 seconds to fall asleep.

上一轮生成结束, 重置prefix cache
2025-08-11 16:12:20 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475097 t:140589927618368: Successfully reset prefix cache
2025-08-11 16:12:21 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475097 t:140589927618368: Sleep mode freed 71.33 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:12:21 executor_base.py:187 INFO:vllm.executor.executor_base p:475097 t:140589927618368: It took 1.705074 seconds to fall asleep.
2025-08-11 16:12:28 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475091 t:139650923292480: Successfully reset prefix cache
2025-08-11 16:12:31 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475091 t:139650923292480: Sleep mode freed 71.32 GiB memory, 4.82 GiB memory is still in use.
2025-08-11 16:12:31 executor_base.py:187 INFO:vllm.executor.executor_base p:475091 t:139650923292480: It took 2.854188 seconds to fall asleep.
2025-08-11 16:12:50 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475098 t:140625347454784: Successfully reset prefix cache
2025-08-11 16:12:53 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475098 t:140625347454784: Sleep mode freed 71.29 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:12:53 executor_base.py:187 INFO:vllm.executor.executor_base p:475098 t:140625347454784: It took 2.869116 seconds to fall asleep.
2025-08-11 16:13:05 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475092 t:140079487534912: Successfully reset prefix cache
2025-08-11 16:13:06 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475092 t:140079487534912: Sleep mode freed 71.33 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:13:06 executor_base.py:187 INFO:vllm.executor.executor_base p:475092 t:140079487534912: It took 1.137816 seconds to fall asleep.

第二轮生成-重置weights和kv_cache
2025-08-11 16:13:10 executor_base.py:203 INFO:vllm.executor.executor_base p:475097 t:140589927618368: It took 1.809626 seconds to wake up tags ['weights'].
2025-08-11 16:13:10 executor_base.py:203 INFO:vllm.executor.executor_base p:475098 t:140625347454784: It took 1.824072 seconds to wake up tags ['weights'].
2025-08-11 16:13:10 executor_base.py:203 INFO:vllm.executor.executor_base p:475096 t:140390354827072: It took 1.801357 seconds to wake up tags ['weights'].
2025-08-11 16:13:10 executor_base.py:203 INFO:vllm.executor.executor_base p:475095 t:140373120079680: It took 1.826042 seconds to wake up tags ['weights'].
2025-08-11 16:13:10 executor_base.py:203 INFO:vllm.executor.executor_base p:475091 t:139650923292480: It took 1.830686 seconds to wake up tags ['weights'].
2025-08-11 16:13:10 executor_base.py:203 INFO:vllm.executor.executor_base p:475093 t:140305750705984: It took 1.831219 seconds to wake up tags ['weights'].
2025-08-11 16:13:10 executor_base.py:203 INFO:vllm.executor.executor_base p:475094 t:140630171514688: It took 1.809485 seconds to wake up tags ['weights'].
2025-08-11 16:13:10 executor_base.py:203 INFO:vllm.executor.executor_base p:475092 t:140079487534912: It took 1.826392 seconds to wake up tags ['weights'].
2025-08-11 16:13:14 executor_base.py:203 INFO:vllm.executor.executor_base p:475091 t:139650923292480: It took 0.232728 seconds to wake up tags ['kv_cache'].
2025-08-11 16:13:14 executor_base.py:203 INFO:vllm.executor.executor_base p:475098 t:140625347454784: It took 0.228185 seconds to wake up tags ['kv_cache'].
2025-08-11 16:13:14 executor_base.py:203 INFO:vllm.executor.executor_base p:475097 t:140589927618368: It took 0.225436 seconds to wake up tags ['kv_cache'].
2025-08-11 16:13:14 executor_base.py:203 INFO:vllm.executor.executor_base p:475092 t:140079487534912: It took 0.175426 seconds to wake up tags ['kv_cache'].
2025-08-11 16:13:14 executor_base.py:203 INFO:vllm.executor.executor_base p:475095 t:140373120079680: It took 0.054448 seconds to wake up tags ['kv_cache'].
2025-08-11 16:13:14 executor_base.py:203 INFO:vllm.executor.executor_base p:475094 t:140630171514688: It took 0.050920 seconds to wake up tags ['kv_cache'].
2025-08-11 16:13:14 executor_base.py:203 INFO:vllm.executor.executor_base p:475093 t:140305750705984: It took 0.049274 seconds to wake up tags ['kv_cache'].
2025-08-11 16:13:14 executor_base.py:203 INFO:vllm.executor.executor_base p:475096 t:140390354827072: It took 0.038036 seconds to wake up tags ['kv_cache'].

第二轮生成

两轮生成结束, 重置prefix cache, sleep
2025-08-11 16:13:37 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475097 t:140589927618368: Successfully reset prefix cache
2025-08-11 16:13:37 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475096 t:140390354827072: Successfully reset prefix cache
2025-08-11 16:13:38 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475095 t:140373120079680: Successfully reset prefix cache
2025-08-11 16:13:40 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475097 t:140589927618368: Sleep mode freed 71.33 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:13:40 executor_base.py:187 INFO:vllm.executor.executor_base p:475097 t:140589927618368: It took 3.636602 seconds to fall asleep.

2025-08-11 16:13:42 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475096 t:140390354827072: Sleep mode freed 71.25 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:13:42 executor_base.py:187 INFO:vllm.executor.executor_base p:475096 t:140390354827072: It took 4.320198 seconds to fall asleep.
2025-08-11 16:13:42 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475095 t:140373120079680: Sleep mode freed 71.24 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:13:42 executor_base.py:187 INFO:vllm.executor.executor_base p:475095 t:140373120079680: It took 3.715163 seconds to fall asleep.
2025-08-11 16:13:53 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475092 t:140079487534912: Successfully reset prefix cache
2025-08-11 16:13:54 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475091 t:139650923292480: Successfully reset prefix cache
2025-08-11 16:13:57 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475092 t:140079487534912: Sleep mode freed 71.31 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:13:57 executor_base.py:187 INFO:vllm.executor.executor_base p:475092 t:140079487534912: It took 4.010950 seconds to fall asleep.
2025-08-11 16:13:57 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475091 t:139650923292480: Sleep mode freed 71.25 GiB memory, 4.82 GiB memory is still in use.
2025-08-11 16:13:57 executor_base.py:187 INFO:vllm.executor.executor_base p:475091 t:139650923292480: It took 2.768709 seconds to fall asleep.
2025-08-11 16:14:16 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475098 t:140625347454784: Successfully reset prefix cache
2025-08-11 16:14:19 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475098 t:140625347454784: Sleep mode freed 71.32 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:14:19 executor_base.py:187 INFO:vllm.executor.executor_base p:475098 t:140625347454784: It took 3.257998 seconds to fall asleep.
2025-08-11 16:14:42 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475094 t:140630171514688: Successfully reset prefix cache
2025-08-11 16:14:45 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475094 t:140630171514688: Sleep mode freed 71.33 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:14:45 executor_base.py:187 INFO:vllm.executor.executor_base p:475094 t:140630171514688: It took 3.264531 seconds to fall asleep.

此处一直在等待140305750705984生成结束 (猜测是有reduce操作)

2025-08-11 16:17:36 block_pool.py:321 INFO:vllm.v1.core.block_pool p:475093 t:140305750705984: Successfully reset prefix cache
2025-08-11 16:17:39 gpu_worker.py:104 INFO:vllm.v1.worker.gpu_worker p:475093 t:140305750705984: Sleep mode freed 71.28 GiB memory, 1.38 GiB memory is still in use.
2025-08-11 16:17:39 executor_base.py:187 INFO:vllm.executor.executor_base p:475093 t:140305750705984: It took 2.676562 seconds to fall asleep.

开始传播

2025-08-11 16:17:42 import_utils.py:87 DEBUG:transformers.utils.import_utils p:475091 t:139650923292480: Detected flash_attn version: 2.8.2
2025-08-11 16:17:42 import_utils.py:87 DEBUG:transformers.utils.import_utils p:475091 t:139650923292480: Detected flash_attn version: 2.8.2

Train:   0%|          | 1/560 [06:58<64:56:20, 418.21s/it]
                                                          
{'loss': 0.00339046, 'grad_norm': 0.02557648, 'learning_rate': 3.57e-06, 'memory(GiB)': 104.33, 'train_speed(iter/s)': 0.001975, 'completions/mean_length': 1988.125, 'completions/min_length': 934.0, 'completions/max_length': 4502.0, 'completions/clipped_ratio': 0.0, 'rewards/Action3F1Reward/mean': 0.68329811, 'rewards/Action3F1Reward/std': 0.38531148, 'rewards/CompletenessReward/mean': 0.2, 'rewards/CompletenessReward/std': 0.0, 'rewards/ActionCountReward/mean': 0.15491357, 'rewards/ActionCountReward/std': 0.07342431, 'rewards/FormatErrorReward/mean': 0.0, 'rewards/FormatErrorReward/std': 0.0, 'reward': 1.0382117, 'reward_std': 0.19291379, 'frac_reward_zero_std': 0.0, 'entropy/mean': 0.0392022, 'entropy/max': 0.09567108, 'entropy/min': 0.02133784, 'entropy/threshold': 0.00133871, 'kl': 0.09781231, 'clip_ratio/low_mean': 0.0, 'clip_ratio/low_min': 0.0, 'clip_ratio/high_mean': 0.0, 'clip_ratio/high_max': 0.0, 'clip_ratio/region_mean': 0.0, 'epoch': 0.04, 'global_step/max_steps': '1/560', 'percentage': '0.18%', 'elapsed_time': '6m 58s', 'remaining_time': '2d 16h 57m 8s'}

Train:   0%|          | 1/560 [06:58<64:56:20, 418.21s/it]
Train:   0%|          | 1/560 [06:58<64:56:20, 418.21s/it]
Train:   0%|          | 2/560 [07:23<28:59:43, 187.07s/it]
Train:   1%|          | 3/560 [07:49<17:35:57, 113.75s/it]
Train:   1%|          | 4/560 [08:16<12:15:26, 79.36s/it] 

训练结束, 下一轮生成

2025-08-11 16:19:51 executor_base.py:203 INFO:vllm.executor.executor_base p:475097 t:140589927618368: It took 3.536754 seconds to wake up tags ['weights'].
2025-08-11 16:19:51 executor_base.py:203 INFO:vllm.executor.executor_base p:475098 t:140625347454784: It took 3.530315 seconds to wake up tags ['weights'].
2025-08-11 16:19:51 executor_base.py:203 INFO:vllm.executor.executor_base p:475093 t:140305750705984: It took 3.551135 seconds to wake up tags ['weights'].
 
``` 

每一轮生成, 都会卡在最后一个, 猜测是有reduce操作

# 尝试-1

在以上实验中, 参数为: 

```
--vllm_gpu_memory_utilization 0.90 \
    --offload_optimizer true \
    --offload_model true \
    --gc_collect_after_offload true \
    --sleep_level 1 \
    --move_model_batches 10

``` 

从日志中查看一轮生成所需的时间: 6m10s

```
Train:   0%|          | 0/560 [00:00<?, ?it/s]
2025-08-11 16:11:32 utils.py:183 DEBUG:vllm.model_executor.models.utils p:475097 t:140589927618368: Loaded weight lm_head.weight with shape torch.Size([151936, 4096])
2025-08-11 16:11:32 utils.py:183 DEBUG:vllm.model_executor.models.utils p:475098 t:140625347454784: Loaded weight lm_head.weight with shape torch.Size([151936, 4096])
...
2025-08-11 16:17:42 import_utils.py:87 DEBUG:transformers.utils.import_utils p:475091 t:139650923292480: Detected flash_attn version: 2.8.2

Train:   0%|          | 1/560 [06:58<64:56:20, 418.21s/it]

``` 

调整参数: 

```
    --vllm_gpu_memory_utilization 0.50 \
    --vllm_tensor_parallel_size 4 \

    --offload_optimizer true \
    --offload_model true \
    --gc_collect_after_offload true \
    --sleep_level 1 \
    --move_model_batches 10
``` 

从日志中查看一轮生成所需的时间: 1m50s

```
Train:   0%|          | 0/560 [00:00<?, ?it/s]
2025-08-11 17:41:35 executor_base.py:203 INFO:vllm.executor.executor_base p:496024 t:139701206603584: It took 0.411399 seconds to wake up tags ['weights'].
2025-08-11 17:41:36 block_pool.py:321 INFO:vllm.v1.core.block_pool p:496024 t:139701206603584: Successfully reset prefix cache
...
2025-08-11 17:43:25 executor_base.py:187 INFO:vllm.executor.executor_base p:496026 t:140491271579456: It took 0.836357 seconds to fall asleep.
Train:   0%|          | 1/560 [02:25<22:38:50, 145.85s/it]
``` 

抽检发现, 这种参数比之前的生成时间都短

GPU使用率: 

![image2025-8-11 17:55:54.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-11%2017%3A55%3A54.png)

# 尝试-2

使用以上参数, 运行时会OOM, 发现单个GPU的内存图如下: 

![image2025-8-12 0:16:2.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-12%200%3A16%3A2.png)

有且只有一个GPU会冒出尖峰, 尖峰周期性出现. 

大模型的解释: 

您好，这张图非常清晰地展示了您所描述的训练过程，并且您观察到的那个突然的尖峰是一个非常典型的现象。结合我们之前分析的训练命令和这张图，我可以非常有把握地告诉您这部分尖峰是什么。

这张图完美地印证了我们之前的分析，并且那个尖峰代表了一个**只在主进程（Rank 0，也就是GPU 0）上执行的周期性任务** 。

### 结论先行

那个在约11.5分钟处冲高到90%以上的**显存尖峰** ，几乎可以肯定是**保存检查点（Checkpoint Saving）**操作导致的。您的命令中设置了 `--save_steps 10`，这意味着每训练10步，系统就会保存一次模型的当前状态。

让我们来详细拆解为什么会发生这种情况，以及为什么只有一条线（很可能是GPU 0）出现了这个尖峰。

### 详细分析

  1. **识别尖峰来源的线索**

     - **周期性** : 这个尖峰不是持续存在的，而是在一个时间点上突然出现，然后迅速回落。这表明它是一个周期性触发的事件。
     - **单一GPU** : 在图中，只有一条浅蓝色的线出现了这个巨大的尖峰，而其他7条线的显存占用保持在它们正常的波动范围内。这强烈暗示该操作是由单个GPU独立完成的。在DDP（分布式数据并行）训练中，保存检查点、评估（Evaluation）和日志记录（Logging）等任务通常被设计为**只在主进程（Rank 0，对应GPU 0）上执行** ，以避免文件写入冲突和冗余计算。
  2. **为什么保存检查点会导致显存剧增？**  
这与您使用的 `--offload_model true` 策略有直接关系。

     - **平时训练时（非尖峰时段）** : 正如我们之前分析的，为了节省显存，模型的大部分权重都被“卸载”到了CPU内存中。在进行前向和反向传播时，每一层模型的权重都像“过客”一样，被从CPU加载到GPU，计算完毕后立刻被踢出GPU，为下一层腾出空间。这使得GPU上任何时刻都只需要存放**少量模型权重 + 累积的激活值** 。这就是您在图中看到的大部分时间里，显存占用在20%到60%之间有规律波动的原因（“悠悠球”效应）。

     - **保存检查点时（尖峰时刻）** : 当需要保存一个完整的、一致的模型检查点时，情况就变了。框架不能再一层一层地处理。它需要在**同一个时间点** 收集到模型**所有** 的状态。对于Rank 0的GPU来说，这个过程通常是：

       1. **暂停训练循环** 。
       2. **在GPU上重新组装模型** : 为了创建一个完整的模型状态字典 (`state_dict`)，框架需要将所有被卸载到CPU的权重层**一次性全部加载回GPU内存中** 。
       3. **内存叠加** : 此刻，Rank 0的GPU内存中**同时存在** ：
          - 训练到一半时留下的**激活值** （Activation）。
          - LoRA参数的**梯度** 。
          - 为了组装而**临时加载回来的全部模型权重** （对于Qwen3-8B，这部分就有约16GB）。
       4. **创建并保存** : 在GPU上创建好完整的状态字典后，将其传输到CPU并写入硬盘。
       5. **释放内存** : 保存操作完成后，临时加载回来的全部模型权重被从GPU内存中释放，GPU显存占用瞬间回落，恢复到正常的训练波动水平。

这个“在GPU上重新组装完整模型”的步骤，就是导致显存占用从正常的50-60%飙升到90%以上的直接原因。它短暂地打破了Offload策略带来的显存节省，是一个必要的、但代价高昂的操作。

### 图表与分析的对应关系

  - **浅蓝色线 (The Spiking GPU)** : 这几乎肯定是 **GPU 0** (Rank 0)。它承担了保存检查点的任务。
  - **其他7条线 (The Stable GPUs)** : 这些是 **GPU 1 到 GPU 7** (Rank 1-7)。在GPU 0忙于保存检查点时，它们处于等待状态，其显存占用保持在训练循环暂停前的水平，或者小幅回落。
  - **W形波动** : 图中所有曲线呈现的“W”或“V”形波动，完美地展示了我们之前讨论的“前向传播（激活值累积，显存上升）-> 反向传播（激活值释放，显存下降）”的循环。

### 总结

您看到的那个尖峰是**一个完全正常的、符合预期的行为** 。它是在使用了激进的CPU Offload策略（`--offload_model true`）后，为了执行保存检查点（`--save_steps 10`）任务而产生的必然结果。

这个现象生动地展示了在资源极限条件下进行大模型训练时，需要在**训练速度** 、**常驻显存** 和**周期性任务开销** 之间做出的权衡。虽然这个尖峰看起来很吓人，但只要它没有导致OOM（Out of Memory）错误，就说明您的配置是成功且有效的。

将offload_model 设置为false, 再观察: 

![image2025-8-12 0:42:44.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-12%200%3A42%3A44.png)

OOM, 还是会有一个尖峰, 原因不明, 认为不符合上面的分析

# 尝试-3

在尝试2的基础上, 调整参数: 

  - offload_model=true
  - per_device_train_batch_size=1

可以完全完成训练, 总时间: 7h 30m 20s

内存使用: 

![image2025-8-12 11:21:39.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-12%2011%3A21%3A39.png)

# 使用pytorch profiler

修改transformers的源文件, 在train loop中增加pytorch profiler调用: 

/data/huangyan/transformers/trainer.py

文件副本: [trainer.py](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/trainer.py)

代码样例:

```
    def _inner_training_loop(
        self, batch_size=None, args=None, resume_from_checkpoint=None, trial=None, ignore_keys_for_eval=None
    ):

	......

	for epoch in range(epochs_trained, num_train_epochs):
		......
		profiler_output_dir = "./profiler_logs"
        os.makedirs(profiler_output_dir, exist_ok=True)
        with torch.profiler.profile(
            activities=[torch.profiler.ProfilerActivity.CPU, torch.profiler.ProfilerActivity.CUDA],
            schedule = torch.profiler.schedule(wait=1, warmup=1, active=2, repeat=1),
            profile_memory=True,
            record_shapes=True,
            with_stack=True,
            on_trace_ready=torch.profiler.tensorboard_trace_handler(profiler_output_dir),
        ) as prof:
		  for _ in range(total_updates):
            ...
            prof.step()
            print(prof.key_averages(group_by_stack_n=5).table(sort_by="cuda_memory_usage", row_limit=15))
        ...
 
``` 

运行后, 会在/data/huangyan/sql-ai-2/profiler_logs中生成文件: 

  - /data/huangyan/sql-ai-2/profiler_logs/[jtkj003_672975.1754980773706849449.pt](<http://jtkj003_672975.1754980773706849449.pt>).trace.json  
/data/huangyan/sql-ai-2/profiler_logs/[jtkj003_672976.1754980773993510788.pt](<http://jtkj003_672976.1754980773993510788.pt>).trace.json  
/data/huangyan/sql-ai-2/profiler_logs/[jtkj003_672977.1754980769362448260.pt](<http://jtkj003_672977.1754980769362448260.pt>).trace.json  
/data/huangyan/sql-ai-2/profiler_logs/[jtkj003_672978.1754980772908095956.pt](<http://jtkj003_672978.1754980772908095956.pt>).trace.json  
/data/huangyan/sql-ai-2/profiler_logs/[jtkj003_672979.1754980769962516433.pt](<http://jtkj003_672979.1754980769962516433.pt>).trace.json  
/data/huangyan/sql-ai-2/profiler_logs/[jtkj003_672980.1754980770738729306.pt](<http://jtkj003_672980.1754980770738729306.pt>).trace.json  
/data/huangyan/sql-ai-2/profiler_logs/[jtkj003_672981.1754980769593689186.pt](<http://jtkj003_672981.1754980769593689186.pt>).trace.json  
/data/huangyan/sql-ai-2/profiler_logs/[jtkj003_672982.1754980773526530317.pt](<http://jtkj003_672982.1754980773526530317.pt>).trace.json

  - 每个GPU对应一个文件

安装tensorboard, 用来打开profiler logs:

```
pip install torch_tb_profiler -i https://pypi.tuna.tsinghua.edu.cn/simple
tensorboard --logdir profiler_logs/
``` 

浏览器打开需要很长时间 (20min左右)

效果: (其中Trace打不开, 浏览器崩溃)

![image2025-8-12 16:8:29.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-12%2016%3A8%3A29.png)

![image2025-8-12 16:8:44.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-12%2016%3A8%3A44.png)

![image2025-8-12 16:8:55.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-12%2016%3A8%3A55.png)

![image2025-8-12 16:9:22.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-12%2016%3A9%3A22.png)

![image2025-8-12 16:10:35.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-12%2016%3A10%3A35.png)

![image2025-8-12 16:11:25.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-12%2016%3A11%3A25.png)

# 解析pytorch profiler的日志

```
(base) root@jtkj003:/data/huangyan/sql-ai-2/profiler_logs# jq 'walk(if type == "array" then .[0] else . end)'  jtkj003_672975.1754980773706849449.pt.trace.json
{
  "schemaVersion": 1,
  "deviceProperties": {
    "id": 0,
    "name": "NVIDIA A100-SXM4-80GB",
    "totalGlobalMem": 84974239744,
    "computeMajor": 8,
    "computeMinor": 0,
    "maxThreadsPerBlock": 1024,
    "maxThreadsPerMultiprocessor": 2048,
    "regsPerBlock": 65536,
    "warpSize": 32,
    "sharedMemPerBlock": 49152,
    "numSms": 108,
    "regsPerMultiprocessor": 65536,
    "sharedMemPerBlockOptin": 166912,
    "sharedMemPerMultiprocessor": 167936
  },
  "cupti_version": 24,
  "cuda_runtime_version": 12060,
  "distributedInfo": {
    "backend": "nccl",
    "rank": 0,
    "world_size": 8,
    "pg_count": 45,
    "pg_config": {
      "pg_name": "0",
      "pg_desc": "default_pg",
      "backend_config": "cuda:nccl",
      "pg_size": 8,
      "ranks": 0
    },
    "nccl_version": "2.26.2"
  },
  "record_shapes": 1,
  "cuda_driver_version": 12040,
  "profile_memory": 1,
  "with_stack": 1,
  "trace_id": "2127972CE10B41BCA327F56A8BC49480",
  "traceEvents": {
    "ph": "X",
    "cat": "user_annotation",
    "name": "ProfilerStep#2",
    "pid": 672975,
    "tid": 672975,
    "ts": 3569824056354.781,
    "dur": 22618996.48,
    "args": {
      "External id": 1,
      "Record function id": 0,
      "finished": true,
      "Ev Idx": 0
    }
  },
  "traceName": "./profiler_logs/jtkj003_672975.1754980773706849449.pt.trace.json",
  "displayTimeUnit": "ms",
  "baseTimeNanoseconds": 1751410836000000000
}
``` 

没看出traceEvents的组织形式, 需要进一步探索, 暂缓

# 使用 torch.cuda.memory._record_memory_history 观察内存分配

代码备份: transformers/trainer.py: [trainer_cuda_memory.py](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/trainer_cuda_memory.py)

代码示例: 

```
def _inner_training_loop(
        self, batch_size=None, args=None, resume_from_checkpoint=None, trial=None, ignore_keys_for_eval=None
    ):
	...
	for epoch in range(epochs_trained, num_train_epochs):
		...
		torch.cuda.memory._record_memory_history(max_entries=500000)
		try:
			......
		    torch.cuda.memory._dump_snapshot('normal_snapshot.pickle')
        except Exception as e:
            print(f"An error occurred: {e}")
            # 即使发生错误，也保存快照以进行事后分析
            torch.cuda.memory._dump_snapshot("oom_snapshot.pickle")
            # 抛出错误，便于上层捕获和处理
            raise

        finally:
            # 停止记录
            torch.cuda.memory._record_memory_history(enabled=None)
``` 

在OOM时, 获取了oom_snapshot.pickle, 将其转换成html: 

```
 python /root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/cuda/_memory_viz.py trace_plot oom_snapshot.pickle -o oom_snapshot.html

``` 

获取了html: [oom_snapshot.html.zip](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/oom_snapshot.html.zip)

![image2025-8-13 10:24:12.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-13%2010%3A24%3A12.png)

可以看到OOM前, 多出的内存部分, 分析样例堆栈, 发现消耗在: 

torch/grpo_trainer.py:1099行: 

![image2025-8-13 12:51:54.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-13%2012%3A51%3A54.png)

大模型的解释: 

非常感谢您提供这段关键的代码！这让我们能够从“推测”进入“精确诊断”阶段。您的代码完全印证了我们之前的分析，并且揭示了更深层次的细节。

我们来逐行解剖这个函数，看看这个14GB的“内存怪兽”是如何诞生的。

### 代码执行流与显存分析

  1. **`logits = model(**model_inputs).logits`**

     - **发生了什么** : 模型执行完整的前向传播，生成`logits`张量。
     - **显存影响** : 这是第一个巨大的显存占用点。`logits`的形状是 `[batch_size, sequence_length, vocab_size]`。根据我们之前的分析，正是在这个步骤，因为输入了一个超长序列（`sequence_length` > 12000），导致这个`logits`张量本身就已经**高达14GB** 。
  2. **`logits = logits[:, :-1, :]`**

     - **发生了什么** : 对`logits`进行切片，移除了最后一个时间步。
     - **显存影响** : 这是个切片操作，在PyTorch中通常会创建一个**视图（View）** ，而不是数据的完整拷贝。它几乎不占用新的显存，只是改变了张量的元数据（形状和步长）。此时`logits`的形状变为 `[batch_size, sequence_length - 1, vocab_size]`。
  3. **`logits = logits[:, -logits_to_keep:, :]`**

     - **发生了什么** : 再次切片，只保留序列末尾的`logits_to_keep`个时间步。`logits_to_keep`通常对应于DPO/GRPO中"chosen"和"rejected"回复的长度。
     - **显存影响** : 同样是一个视图操作，不产生新的显存开销。此时`logits`的形状变为 `[batch_size, logits_to_keep, vocab_size]`。
     - **关键洞察** : 结合我们对14GB大小的估算（`batch_size * logits_to_keep ≈ 50,110`），这表明您的数据中存在**回复长度（`logits_to_keep`）本身就超过12000**的样本。这比仅仅是输入上下文长要严重得多。
  4. **`logits = logits / self.temperature` (第1119行 - OOM引爆点)**

     - **发生了什么** : 对这个已经存在的、14GB大小的`logits`张量执行除法操作。
     - **显存影响** : **这是最致命的一步！** 在Python和PyTorch中，`A = B / C` 这样的写法是一个**“out-of-place”（非原地）**操作。它的执行逻辑是：  
a. 为运算结果（等号右边）分配一块**全新的、和`logits`一样大的内存空间**。这就是堆栈中那个**14GB的`allocation`请求**。  
b. 执行除法运算，并将结果存入这块新分配的空间。  
c. 将变量名 `logits` 指向这块新的内存地址。
     - **OOM的原因** : 在执行a步骤时，GPU中已经存在一个14GB的`logits`张量了。现在它需要**额外再找一块14GB的连续空间** 。此时，GPU的总需求变成了 `(原来的14GB logits) + (新的14GB结果) + (模型参数和其他激活值)`。这个需求瞬间超出了GPU的物理极限，导致分配失败，程序崩溃。

### 结论与“罪责”划分

  - **根本原因（The Root Cause）** : 您的数据集中存在**回复（completion）部分极长** 的样本，导致`logits_to_keep`的值异常巨大，从而在模型前向传播后产生了一个14GB的`logits`张量。
  - **直接原因（The Trigger）** : `logits = logits / self.temperature` 这行代码的**“out-of-place”**性质。它试图为已经巨大的张量再创建一个同样巨大的副本，成为了压垮骆驼的最后一根稻草。

### 解决方案

现在我们有了代码，可以提出更具体的解决方案：

  1. **【治本】数据层面：严格限制回复长度**  
这是最根本、最有效的解决方案。您必须在数据预处理阶段，检查并**截断（truncate）**所有样本的回复部分，确保其长度在一个合理的范围内（例如，对于大多数任务，不超过2048或4096）。这会直接让`logits_to_keep`变小，从源头上阻止14GB张量的产生。

  2. **【治标】代码层面：使用原地（In-place）操作**  
这是一个可以立即尝试的、缓解问题的代码修改。将“out-of-place”操作改为“in-place”操作。

**修改前** :
         
         logits = logits / self.temperature
         

**修改后 (任选一种)** :
         
         # 方案一：使用原地除法函数
         logits.div_(self.temperature)
         
         # 方案二：使用原地操作符
         logits /= self.temperature 
         

**效果** : 原地操作会直接在`logits`张量**自身的内存上** 进行修改，而**不会申请新的内存空间** 。这样就避免了那个致命的14GB内存分配请求。

**请注意** : 这个“治标”方法虽然能避免此次OOM，但它并没有解决14GB `logits`张量本身存在于显存中的问题。如果您的GPU连容纳单个14GB张量都非常勉强，那么在其他地方（如反向传播时）仍有可能OOM。因此，**强烈建议将此方法与方案一（数据截断）结合使用** 。

修改代码为: 

```
logits.div_(self.temperature)
``` 

继续测试, OOM后, 发现消耗在--log_entropy 的处理, 将其设置为false

继续测试, OOM: 

  - GPU
    - ![image2025-8-13 12:57:30.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-13%2012%3A57%3A30.png)
  - torch.CUDA 分配图: 未见异常
    - ![image2025-8-13 12:58:9.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-13%2012%3A58%3A9.png)
  - 错误日志: 

```
[rank1]:[E813 12:25:30.691567129 ProcessGroupNCCL.cpp:632] [Rank 1] Watchdog caught collective operation timeout: WorkNCCL(SeqNum=57, OpType=ALLGATHER, NumelIn=1, NumelOut=4, Timeout(ms)=600000) ran for 600038 milliseconds before timing out.
[rank1]:[E813 12:25:30.692163833 ProcessGroupNCCL.cpp:2271] [PG ID 1 PG GUID 1 Rank 1]  failure detected by watchdog at work sequence id: 57 PG status: last enqueued work: 57, last completed work: 56
[rank1]:[E813 12:25:30.692194747 ProcessGroupNCCL.cpp:670] Stack trace of the failed collective not found, potentially because FlightRecorder is disabled. You can enable it by setting TORCH_NCCL_TRACE_BUFFER_SIZE to a non-zero value.
[rank0]:[E813 12:25:30.947799196 ProcessGroupNCCL.cpp:632] [Rank 0] Watchdog caught collective operation timeout: WorkNCCL(SeqNum=57, OpType=ALLGATHER, NumelIn=1, NumelOut=4, Timeout(ms)=600000) ran for 600096 milliseconds before timing out.
[rank0]:[E813 12:25:30.948355712 ProcessGroupNCCL.cpp:2271] [PG ID 1 PG GUID 1 Rank 0]  failure detected by watchdog at work sequence id: 57 PG status: last enqueued work: 57, last completed work: 56
[rank0]:[E813 12:25:30.948386260 ProcessGroupNCCL.cpp:670] Stack trace of the failed collective not found, potentially because FlightRecorder is disabled. You can enable it by setting TORCH_NCCL_TRACE_BUFFER_SIZE to a non-zero value.
[rank0]:[E813 12:25:30.948459379 ProcessGroupNCCL.cpp:2106] [PG ID 1 PG GUID 1 Rank 0] First PG on this rank to signal dumping.
[rank0]:[E813 12:25:31.213133402 ProcessGroupNCCL.cpp:1746] [PG ID 0 PG GUID 0(default_pg) Rank 0] Received a dump signal due to a collective timeout from this local rank and we will try our best to dump the debug info. Last enqueued NCCL work: 593, last completed NCCL work: 593.This is most likely caused by incorrect usages of collectives, e.g., wrong sizes used across ranks, the order of collectives is not same for all ranks or the scheduled collective, for some reason, didn't run. Additionally, this can be caused by GIL deadlock or other reasons such as network errors or bugs in the communications library (e.g. NCCL), etc. 
[rank0]:[E813 12:25:31.213316968 ProcessGroupNCCL.cpp:1536] [PG ID 0 PG GUID 0(default_pg) Rank 0] ProcessGroupNCCL preparing to dump debug info. Include stack trace: 1
An error occurred: CUDA out of memory. Tried to allocate 184.13 GiB. GPU 2 has a total capacity of 79.14 GiB of which 37.02 GiB is free. Including non-PyTorch memory, this process has 42.05 GiB memory in use. Of the allocated memory 38.58 GiB is allocated by PyTorch, with 105.08 MiB allocated in private pools (e.g., CUDA Graphs), and 850.91 MiB is reserved by PyTorch but unallocated. If reserved but unallocated memory is large try setting PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True to avoid fragmentation.  See documentation for Memory Management  (https://pytorch.org/docs/stable/notes/cuda.html#environment-variables)
An error occurred: CUDA out of memory. Tried to allocate 184.13 GiB. GPU 1 has a total capacity of 79.14 GiB of which 37.80 GiB is free. Including non-PyTorch memory, this process has 41.28 GiB memory in use. Of the allocated memory 38.58 GiB is allocated by PyTorch, with 105.08 MiB allocated in private pools (e.g., CUDA Graphs), and 52.95 MiB is reserved by PyTorch but unallocated. If reserved but unallocated memory is large try setting PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True to avoid fragmentation.  See documentation for Memory Management  (https://pytorch.org/docs/stable/notes/cuda.html#environment-variables)
[rank2]: Traceback (most recent call last):
[rank2]:   File "/data/huangyan/ms-swift/swift/cli/rlhf.py", line 5, in <module>
[rank2]:     rlhf_main()
[rank2]:   File "/data/huangyan/ms-swift/swift/llm/train/rlhf.py", line 183, in rlhf_main
[rank2]:     return SwiftRLHF(args).main()
[rank2]:   File "/data/huangyan/ms-swift/swift/llm/base.py", line 49, in main
[rank2]:     result = self.run()
[rank2]:   File "/data/huangyan/ms-swift/swift/llm/train/sft.py", line 259, in run
[rank2]:     return self.train(trainer)
[rank2]:   File "/data/huangyan/ms-swift/swift/llm/train/sft.py", line 320, in train
[rank2]:     trainer.train(trainer.args.resume_from_checkpoint)
[rank2]:   File "/data/huangyan/ms-swift/swift/trainers/mixin.py", line 675, in train
[rank2]:     res = super().train(*args, **kwargs)
[rank2]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 2238, in train
[rank2]:     return inner_training_loop(
[rank2]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 2585, in _inner_training_loop
[rank2]:     tr_loss_step = self.training_step(model, inputs, num_items_in_batch)
[rank2]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 1561, in training_step
[rank2]:     loss = super().training_step(model, inputs, num_items_in_batch)
[rank2]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 3805, in training_step
[rank2]:     inputs = self._prepare_inputs(inputs)
[rank2]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/utils.py", line 168, in wrapper
[rank2]:     return func(self, *args, **kwargs)
[rank2]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 411, in _prepare_inputs
[rank2]:     generation_batch = self._generate_and_score_completions(generation_batch)
[rank2]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 961, in _generate_and_score_completions
[rank2]:     inputs = self._generate_completions(inputs)
[rank2]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 932, in _generate_completions
[rank2]:     inputs, outputs = self._fast_infer(inputs)
[rank2]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 909, in _fast_infer
[rank2]:     outputs = self._infer_single_or_multi_turn(inputs, self.request_config)
[rank2]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 728, in _infer_single_or_multi_turn
[rank2]:     results: List[ChatCompletionResponse] = self._infer(inputs, request_config, is_global_inputs)
[rank2]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 657, in _infer
[rank2]:     torch.distributed.all_gather_object(all_input_lengths, local_input_length, group=self.tp_group)
[rank2]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/distributed/c10d_logger.py", line 81, in wrapper
[rank2]:     return func(*args, **kwargs)
[rank2]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/distributed/distributed_c10d.py", line 3042, in all_gather_object
[rank2]:     input_tensor.resize_(max_object_size)
[rank2]: torch.OutOfMemoryError: CUDA out of memory. Tried to allocate 184.13 GiB. GPU 2 has a total capacity of 79.14 GiB of which 37.02 GiB is free. Including non-PyTorch memory, this process has 42.05 GiB memory in use. Of the allocated memory 38.58 GiB is allocated by PyTorch, with 105.08 MiB allocated in private pools (e.g., CUDA Graphs), and 850.91 MiB is reserved by PyTorch but unallocated. If reserved but unallocated memory is large try setting PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True to avoid fragmentation.  See documentation for Memory Management  (https://pytorch.org/docs/stable/notes/cuda.html#environment-variables)
[rank1]: Traceback (most recent call last):
[rank1]:   File "/data/huangyan/ms-swift/swift/cli/rlhf.py", line 5, in <module>
[rank1]:     rlhf_main()
[rank1]:   File "/data/huangyan/ms-swift/swift/llm/train/rlhf.py", line 183, in rlhf_main
[rank1]:     return SwiftRLHF(args).main()
[rank1]:   File "/data/huangyan/ms-swift/swift/llm/base.py", line 49, in main
[rank1]:     result = self.run()
[rank1]:   File "/data/huangyan/ms-swift/swift/llm/train/sft.py", line 259, in run
[rank1]:     return self.train(trainer)
[rank1]:   File "/data/huangyan/ms-swift/swift/llm/train/sft.py", line 320, in train
[rank1]:     trainer.train(trainer.args.resume_from_checkpoint)
[rank1]:   File "/data/huangyan/ms-swift/swift/trainers/mixin.py", line 675, in train
[rank1]:     res = super().train(*args, **kwargs)
[rank1]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 2238, in train
[rank1]:     return inner_training_loop(
[rank1]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 2585, in _inner_training_loop
[rank1]:     tr_loss_step = self.training_step(model, inputs, num_items_in_batch)
[rank1]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 1561, in training_step
[rank1]:     loss = super().training_step(model, inputs, num_items_in_batch)
[rank1]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 3805, in training_step
[rank1]:     inputs = self._prepare_inputs(inputs)
[rank1]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/utils.py", line 168, in wrapper
[rank1]:     return func(self, *args, **kwargs)
[rank1]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 411, in _prepare_inputs
[rank1]:     generation_batch = self._generate_and_score_completions(generation_batch)
[rank1]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 961, in _generate_and_score_completions
[rank1]:     inputs = self._generate_completions(inputs)
[rank1]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 932, in _generate_completions
[rank1]:     inputs, outputs = self._fast_infer(inputs)
[rank1]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 909, in _fast_infer
[rank1]:     outputs = self._infer_single_or_multi_turn(inputs, self.request_config)
[rank1]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 728, in _infer_single_or_multi_turn
[rank1]:     results: List[ChatCompletionResponse] = self._infer(inputs, request_config, is_global_inputs)
[rank1]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 657, in _infer
[rank1]:     torch.distributed.all_gather_object(all_input_lengths, local_input_length, group=self.tp_group)
[rank1]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/distributed/c10d_logger.py", line 81, in wrapper
[rank1]:     return func(*args, **kwargs)
[rank1]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/distributed/distributed_c10d.py", line 3042, in all_gather_object
[rank1]:     input_tensor.resize_(max_object_size)
[rank1]: torch.OutOfMemoryError: CUDA out of memory. Tried to allocate 184.13 GiB. GPU 1 has a total capacity of 79.14 GiB of which 37.80 GiB is free. Including non-PyTorch memory, this process has 41.28 GiB memory in use. Of the allocated memory 38.58 GiB is allocated by PyTorch, with 105.08 MiB allocated in private pools (e.g., CUDA Graphs), and 52.95 MiB is reserved by PyTorch but unallocated. If reserved but unallocated memory is large try setting PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True to avoid fragmentation.  See documentation for Memory Management  (https://pytorch.org/docs/stable/notes/cuda.html#environment-variables)
[rank3]:[E813 12:26:00.651268916 ProcessGroupNCCL.cpp:1321] [PG ID 1 PG GUID 1 Rank 3] Future for ProcessGroup abort timed out after 600000 ms
[rank2]:[W813 12:26:30.256219309 ProcessGroupNCCL.cpp:2114] [PG ID 1 PG GUID 1 Rank 2] timed out after waiting for 60000ms flight recorder dumps to finish.
[rank2]:[E813 12:26:30.256393068 ProcessGroupNCCL.cpp:684] [Rank 2] Some NCCL operations have failed or timed out. Due to the asynchronous nature of CUDA kernels, subsequent GPU operations might run on corrupted/incomplete data.
[rank2]:[E813 12:26:30.256416041 ProcessGroupNCCL.cpp:698] [Rank 2] To avoid data inconsistency, we are taking the entire process down.

```

    - 要分配一个超大的内存块: Tried to allocate 184.13 GiB, 进行all_gather?
    - 增加调试日志, 但未有结论

继续测试, 出现 OOM, 堆栈如下: 

```
168373 Addr: b'7f99c4000000_0, Size: 16.2GiB (17403357184 bytes) allocation, Total memory used after allocation: 87.7GiB (94120691490 bytes), Compile context: None, timestamp Wed Aug 13 2025 13:21:04 GMT+0800 (中国标准时间)
CUDACachingAllocator.cpp:0:c10::cuda::CUDACachingAllocator::Native::DeviceCachingAllocator::malloc(signed char, unsigned long, CUstream_st*)
:0:c10::cuda::CUDACachingAllocator::Native::NativeCachingAllocator::malloc(void**, signed char, unsigned long, CUstream_st*)
:0:c10::cuda::CUDACachingAllocator::Native::NativeCachingAllocator::allocate(unsigned long)
:0:c10::StorageImpl::StorageImpl(c10::StorageImpl::use_byte_size_t, c10::SymInt const&, c10::Allocator*, bool)
??:0:at::detail::empty_strided_generic(c10::ArrayRef<long>, c10::ArrayRef<long>, c10::Allocator*, c10::DispatchKeySet, c10::ScalarType)
??:0:at::detail::empty_strided_cuda(c10::ArrayRef<long>, c10::ArrayRef<long>, c10::ScalarType, std::optional<c10::Device>)
??:0:at::detail::empty_strided_cuda(c10::ArrayRef<long>, c10::ArrayRef<long>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>)
??:0:at::native::empty_strided_cuda(c10::ArrayRef<long>, c10::ArrayRef<long>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>)
RegisterCUDA_0.cpp:0:at::(anonymous namespace)::(anonymous namespace)::wrapper_CUDA__empty_strided(c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>)
RegisterCUDA_0.cpp:0:c10::impl::wrap_kernel_functor_unboxed_<c10::impl::detail::WrapFunctionIntoFunctor_<c10::CompileTimeFunctionPointer<at::Tensor (c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>), &at::(anonymous namespace)::(anonymous namespace)::wrapper_CUDA__empty_strided>, at::Tensor, c10::guts::typelist::typelist<c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool> > >, at::Tensor (c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>)>::call(c10::OperatorKernel*, c10::DispatchKeySet, c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>)
??:0:at::_ops::empty_strided::redispatch(c10::DispatchKeySet, c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>)
RegisterBackendSelect.cpp:0:c10::impl::wrap_kernel_functor_unboxed_<c10::impl::detail::WrapFunctionIntoFunctor_<c10::CompileTimeFunctionPointer<at::Tensor (c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>), &at::(anonymous namespace)::empty_strided>, at::Tensor, c10::guts::typelist::typelist<c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool> > >, at::Tensor (c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>)>::call(c10::OperatorKernel*, c10::DispatchKeySet, c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>)
??:0:at::_ops::empty_strided::call(c10::ArrayRef<c10::SymInt>, c10::ArrayRef<c10::SymInt>, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>)
:0:at::empty_strided(c10::ArrayRef<long>, c10::ArrayRef<long>, c10::TensorOptions)
??:0:at::native::_to_copy(at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)
RegisterCompositeExplicitAutograd_0.cpp:0:c10::impl::wrap_kernel_functor_unboxed_<c10::impl::detail::WrapFunctionIntoFunctor_<c10::CompileTimeFunctionPointer<at::Tensor (at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>), &at::(anonymous namespace)::(anonymous namespace)::wrapper_CompositeExplicitAutograd___to_copy>, at::Tensor, c10::guts::typelist::typelist<at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat> > >, at::Tensor (at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)>::call(c10::OperatorKernel*, c10::DispatchKeySet, at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)
??:0:at::_ops::_to_copy::redispatch(c10::DispatchKeySet, at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)
RegisterBackendSelect.cpp:0:c10::impl::wrap_kernel_functor_unboxed_<c10::impl::detail::WrapFunctionIntoFunctor_<c10::CompileTimeFunctionPointer<at::Tensor (at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>), &at::(anonymous namespace)::_to_copy>, at::Tensor, c10::guts::typelist::typelist<at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat> > >, at::Tensor (at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)>::call(c10::OperatorKernel*, c10::DispatchKeySet, at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)
??:0:at::_ops::_to_copy::redispatch(c10::DispatchKeySet, at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)
VariableType_0.cpp:0:torch::autograd::VariableType::(anonymous namespace)::_to_copy(c10::DispatchKeySet, at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)
VariableType_0.cpp:0:c10::impl::wrap_kernel_functor_unboxed_<c10::impl::detail::WrapFunctionIntoFunctor_<c10::CompileTimeFunctionPointer<at::Tensor (c10::DispatchKeySet, at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>), &torch::autograd::VariableType::(anonymous namespace)::_to_copy>, at::Tensor, c10::guts::typelist::typelist<c10::DispatchKeySet, at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat> > >, at::Tensor (c10::DispatchKeySet, at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)>::call(c10::OperatorKernel*, c10::DispatchKeySet, at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)
??:0:at::_ops::_to_copy::call(at::Tensor const&, std::optional<c10::ScalarType>, std::optional<c10::Layout>, std::optional<c10::Device>, std::optional<bool>, bool, std::optional<c10::MemoryFormat>)
??:0:at::native::to(at::Tensor const&, c10::ScalarType, bool, bool, std::optional<c10::MemoryFormat>)
RegisterCompositeImplicitAutograd_0.cpp:0:c10::impl::wrap_kernel_functor_unboxed_<c10::impl::detail::WrapFunctionIntoFunctor_<c10::CompileTimeFunctionPointer<at::Tensor (at::Tensor const&, c10::ScalarType, bool, bool, std::optional<c10::MemoryFormat>), &at::(anonymous namespace)::(anonymous namespace)::wrapper_CompositeImplicitAutograd_dtype_to>, at::Tensor, c10::guts::typelist::typelist<at::Tensor const&, c10::ScalarType, bool, bool, std::optional<c10::MemoryFormat> > >, at::Tensor (at::Tensor const&, c10::ScalarType, bool, bool, std::optional<c10::MemoryFormat>)>::call(c10::OperatorKernel*, c10::DispatchKeySet, at::Tensor const&, c10::ScalarType, bool, bool, std::optional<c10::MemoryFormat>)
??:0:at::_ops::to_dtype::call(at::Tensor const&, c10::ScalarType, bool, bool, std::optional<c10::MemoryFormat>)
python_variable_methods.cpp:0:torch::autograd::THPVariable_to_type(_object*, c10::ScalarType, std::optional<c10::MemoryFormat>)
python_variable_methods.cpp:0:torch::autograd::THPVariable_float(_object*, _object*, _object*)
/usr/local/src/conda/python-3.10.18/Objects/descrobject.c:344:method_vectorcall_VARARGS_KEYWORDS
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/accelerate/utils/operations.py:777:_convert_to_fp32
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/accelerate/utils/operations.py:126:recursively_apply
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/accelerate/utils/operations.py:119:<dictcomp>
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/accelerate/utils/operations.py:118:recursively_apply
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/accelerate/utils/operations.py:785:convert_to_fp32
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/accelerate/utils/operations.py:806:__call__
/usr/local/src/conda/python-3.10.18/Objects/typeobject.c:7494:slot_tp_call
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/accelerate/utils/operations.py:818:forward
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/nn/modules/module.py:1762:_call_impl
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/nn/modules/module.py:1751:_wrapped_call_impl
/usr/local/src/conda/python-3.10.18/Objects/typeobject.c:7494:slot_tp_call
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/nn/parallel/distributed.py:1464:_run_ddp_forward
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/nn/parallel/distributed.py:1637:forward
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/nn/modules/module.py:1762:_call_impl
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/nn/modules/module.py:1751:_wrapped_call_impl
/usr/local/src/conda/python-3.10.18/Objects/typeobject.c:7494:slot_tp_call
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/trl/trainer/grpo_trainer.py:1112:_get_per_token_logps_and_entropies
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/trl/extras/profiling.py:98:wrapper
/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py:1458:_get_per_token_logps_and_entropies
/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/utils.py:168:wrapper
/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py:1255:_compute_loss
/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py:1247:compute_loss
/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/utils.py:168:wrapper
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py:3811:training_step
/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py:1563:training_step
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py:2585:_inner_training_loop
/usr/local/src/conda/python-3.10.18/Modules/_functoolsmodule.c:323:partial_call
/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py:2238:train
/data/huangyan/ms-swift/swift/trainers/mixin.py:675:train
/data/huangyan/ms-swift/swift/llm/train/sft.py:320:train
/data/huangyan/ms-swift/swift/llm/train/sft.py:259:run
/data/huangyan/ms-swift/swift/llm/base.py:49:main
``` 

  - 大模型的分析: 
    - 好的，我们来对这段新的堆栈信息进行一次彻底的分析。这个堆栈再次揭示了一个由巨型张量引发的OOM，但这次的“引爆点”与之前的都不同，它暴露了混合精度训练中的一个关键环节。

### 核心观察与初步结论

      1. **惊人的内存申请** : `Size: 16.2GiB (17403357184 bytes) allocation`。

         - 您的程序在崩溃瞬间，试图一次性申请**高达16.2GB** 的GPU显存。这比我们之前看到的14GB还要大。
      2. **关键的Python代码** : `accelerate/utils/operations.py:777:_convert_to_fp32`。

         - 这是本次OOM在Python代码中的“震中”。函数名`_convert_to_fp32`的意图非常明确：**将一个张量的数据类型转换为32位浮点数（FP32）** 。在PyTorch中，这通常对应于调用 `.float()` 或 `.to(torch.float32)`。
      3. **关键的C++操作** : `at::native::_to_copy` -> `at::empty_strided`。

         - 这揭示了OOM的底层机制。当您执行类型转换时，PyTorch并不是在原地修改数据，而是执行一个“拷贝”操作：  
a. 首先，调用 `empty_strided` 创建一个**全新的、空的张量** ，其尺寸与原张量相同，但数据类型是新的目标类型（FP32）。  
b. 然后，将原张量的数据拷贝并转换到这个新张量中。
         - 您的程序在执行步骤 a 时崩溃，因为创建这个新的空张量所需的16.2GB空间无法被满足。

**初步结论** ：这次OOM的直接原因是，代码试图将一个**已经非常巨大（约8.1GB）的张量** 从低精度（如FP16或BF16）转换为全精度（FP32）。这个转换操作需要**双倍的内存空间** ，从而触发了致命的16.2GB内存分配请求。

* * *

### 堆栈的详细“故事线”分析

我们从下到上，一步步还原事件的全貌。

#### 阶段一：GRPO训练与损失计算 (与上次类似)

      - `.../sft.py -> .../trainer.py -> .../grpo_trainer.py:training_step -> ...:compute_loss -> ...:_get_per_token_logps_and_entropies`
        - **发生了什么** : 您的训练流程进入了GRPO的损失计算部分，并调用了核心函数 `_get_per_token_logps_and_entropies`。这与我们之前分析的一个场景非常相似。

#### 阶段二：DDP与Accelerate介入

      - `/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/torch/nn/parallel/distributed.py:1637:forward`
        - **发生了什么** : 这表明您的模型被`torch.nn.parallel.DistributedDataParallel` (DDP) 所包裹。这意味着您正在进行多GPU数据并行训练。
      - `/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/accelerate/utils/operations.py:818:forward`
        - **发生了什么** : 在DDP的`forward`调用链中，出现了`accelerate`库的`operations.py`。这说明`accelerate`正在管理您的训练循环和数据类型。

#### 阶段三：引爆点 - 数据类型转换

      - `/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/accelerate/utils/operations.py:785:convert_to_fp32`
      - `/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/accelerate/utils/operations.py:777:_convert_to_fp32`
      - `python_variable_methods.cpp:0:torch::autograd::THPVariable_float`
        - **发生了什么** : `accelerate`库决定将某个张量转换为FP32。在混合精度训练中，这是一种常见策略。通常，模型的大部分计算（如矩阵乘法）使用FP16或BF16以节省显存和加速计算，但在计算最终的损失函数（Loss）时，会**将模型的输出（logits）转换回FP32** ，因为FP32的数值范围更广、精度更高，可以避免梯度下溢或上溢，使损失计算和反向传播更稳定。

#### 阶段四：从Python `.float()` 到C++内存分配失败

      - `??:0:at::_ops::to_dtype::call` -> `??:0:at::native::_to_copy` -> `??:0:at::detail::empty_strided_cuda`
        - **发生了什么** : Python的`.float()`调用被传递到PyTorch的C++后端。后端将其实现为一个“拷贝”操作。
      - **`168373 Addr: ... Size: 16.2GiB ... allocation`**
        - **最终的失败** : PyTorch尝试创建一个16.2GB的空张量来存放转换后的结果，但GPU没有足够的连续空闲内存，分配失败，OOM。

* * *

### 核心问题综合分析：为何需要16.2GB？

这个数字完美地揭示了问题的本质。

      1. **混合精度训练** : 您的模型很可能在`bfloat16`或`float16`模式下运行。在这种模式下，一个浮点数占用**2个字节** 。
      2. **FP32转换** : `_convert_to_fp32`将数据转换为`float32`模式。在这种模式下，一个浮点数占用**4个字节** 。
      3. **内存加倍** : 因此，将一个张量从BF16/FP16转换为FP32，其占用的内存会**精确地翻倍** 。
      4. **反推原张量大小** :
         - 目标张量大小: `16.2 GiB`
         - 原张量大小: `16.2 GiB / 2 = 8.1 GiB`

**结论非常清晰** ：  
在OOM发生前，您的GPU中已经存在一个**大小约为8.1GB的`logits`张量**（以BF16/FP16格式存储）。当`accelerate`为了稳定地计算损失而尝试将其转换为FP32时，它需要申请一个`8.1 * 2 = 16.2 GB`的新空间。这个操作直接导致了显存崩溃。

而一个8.1GB的`logits`张量，再次指向了我们反复遇到的根本原因：**您的模型正在处理一个包含极端长度序列的批次** 。

### 解决方案

这次的堆栈信息让我们对问题有了更全面的认识，解决方案也更加立体：

      1. **【治本之策】处理超长序列** :  
这依然是**最根本、最重要** 的解决方案。您必须在数据预处理阶段，检查并**截断（truncate）**所有样本的 `prompt` 和 `completion` 部分，确保 `prompt_length + completion_length` 的总和在一个可控的最大值（如2048, 4096, 8192）之内。只有这样，才能从源头上阻止巨型`logits`张量的产生。

      2. **【检查配置】审视混合精度策略** :  
检查您的`Trainer`或`accelerate`配置。虽然在计算loss时将logits转为FP32是标准做法，但您需要了解它正在发生。

         - 确认您是否确实开启了`fp16`或`bf16`混合精度训练。
         - 这个转换通常是自动且有益的，不建议强行关闭它，因为这可能导致训练不稳定。关键还是解决输入张量过大的问题。
      3. **【缓解措施】减小批次大小** :  
如果由于某种原因必须处理较长的序列，那么唯一的办法就是进一步减小`per_device_train_batch_size`，比如降到1。这会等比例地减小`logits`张量的大小，从而降低FP32转换后的内存峰值。同时配合梯度累积（`gradient_accumulation_steps`）来维持等效的总批次大小。

总而言之，这次的OOM是“**超长序列** ”这一根本原因，在“**混合精度训练的类型转换** ”这一特定场景下的又一次爆发。您需要集中精力解决数据集中存在的超长样本问题。

继续测试, 经过一小时后, 发生OOM:

  - commit: f461ebb63856ae0e49255bd749c2d61bc8ae275d
  - ![image2025-8-13 17:15:56.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-13%2017%3A15%3A56.png)
  - torch.cuda 监控图: 
    - ![image2025-8-13 17:16:21.png](/assets/01KJBZX5SBH11JY81B6NSAW0Y6/image2025-8-13%2017%3A16%3A21.png)
    - 未见异常
  - 堆栈: 

```
CUDA Error: out of memory at /workspace/csrc/cumem_allocator.cpp:62
An error occurred: CUDA Error: out of memory at /workspace/csrc/cumem_allocator.cpp:62
[rank6]: Traceback (most recent call last):
[rank6]:   File "/data/huangyan/ms-swift/swift/cli/rlhf.py", line 5, in <module>
[rank6]:     rlhf_main()
[rank6]:   File "/data/huangyan/ms-swift/swift/llm/train/rlhf.py", line 183, in rlhf_main
[rank6]:     return SwiftRLHF(args).main()
[rank6]:   File "/data/huangyan/ms-swift/swift/llm/base.py", line 49, in main
[rank6]:     result = self.run()
[rank6]:   File "/data/huangyan/ms-swift/swift/llm/train/sft.py", line 259, in run
[rank6]:     return self.train(trainer)
[rank6]:   File "/data/huangyan/ms-swift/swift/llm/train/sft.py", line 320, in train
[rank6]:     trainer.train(trainer.args.resume_from_checkpoint)
[rank6]:   File "/data/huangyan/ms-swift/swift/trainers/mixin.py", line 675, in train
[rank6]:     res = super().train(*args, **kwargs)
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 2238, in train
[rank6]:     return inner_training_loop(
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 2585, in _inner_training_loop
[rank6]:     tr_loss_step = self.training_step(model, inputs, num_items_in_batch)
[rank6]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 1563, in training_step
[rank6]:     loss = super().training_step(model, inputs, num_items_in_batch)
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/transformers/trainer.py", line 3805, in training_step
[rank6]:     inputs = self._prepare_inputs(inputs)
[rank6]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/utils.py", line 168, in wrapper
[rank6]:     return func(self, *args, **kwargs)
[rank6]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 411, in _prepare_inputs
[rank6]:     generation_batch = self._generate_and_score_completions(generation_batch)
[rank6]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 963, in _generate_and_score_completions
[rank6]:     inputs = self._generate_completions(inputs)
[rank6]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 934, in _generate_completions
[rank6]:     inputs, outputs = self._fast_infer(inputs)
[rank6]:   File "/data/huangyan/ms-swift/swift/trainers/rlhf_trainer/grpo_trainer.py", line 875, in _fast_infer
[rank6]:     self.engine.engine.wake_up(tags=['weights'])
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/v1/engine/llm_engine.py", line 281, in wake_up
[rank6]:     self.engine_core.wake_up(tags)
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/v1/engine/core_client.py", line 264, in wake_up
[rank6]:     self.engine_core.wake_up(tags)
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/v1/engine/core.py", line 353, in wake_up
[rank6]:     self.model_executor.wake_up(tags)
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/executor/executor_base.py", line 201, in wake_up
[rank6]:     self.collective_rpc("wake_up", kwargs=dict(tags=tags))
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/executor/uniproc_executor.py", line 58, in collective_rpc
[rank6]:     answer = run_method(self.driver_worker, method, args, kwargs)
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/utils/__init__.py", line 2985, in run_method
[rank6]:     return func(*args, **kwargs)
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/v1/worker/gpu_worker.py", line 113, in wake_up
[rank6]:     allocator.wake_up(tags)
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/device_allocator/cumem.py", line 225, in wake_up
[rank6]:     create_and_map(handle)
[rank6]:   File "/root/anaconda3/envs/huangyan/lib/python3.10/site-packages/vllm/device_allocator/cumem.py", line 78, in create_and_map
[rank6]:     python_create_and_map(*allocation_handle)
[rank6]: RuntimeError: CUDA Error: out of memory at /workspace/csrc/cumem_allocator.cpp:62

```

  
分析: 应当是VLLM的 "持有内存"爆掉

    - VLLM声明持有了gpu_utilization_ratio=0.5的内存, 但实际未使用那么多. torch使用的内存超过0.5的边界, 并无法释放, 当vllm在wake_up时, 发现实际的内存不足, 导致OOM
