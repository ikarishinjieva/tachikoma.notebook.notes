---
title: 20250729 - 对GRPO的completion文件和step/epoch计算的理解
confluence_page_id: 4161763
created_at: 2025-07-29T08:17:19+00:00
updated_at: 2025-07-29T08:17:19+00:00
---

# 对completion文件的理解

completion文件, 只包括主进程的数据. 如果有8个GPU, 那么其中只包括1/8的数据. 

# 对于step的计算

<https://github.com/modelscope/ms-swift/blob/d6219459c89fdcdceb7d78a15a7180f4a0391f53/swift/plugin/optimizer.py#L18>

GRPO的总任务数 = 数据量 * num_generation

每个step需要处理的任务量 = per_device_train_batch_size * gradient_accumulation_steps * world_size (GPU数量)

每个epoch包含的step数量 = GRPO的总任务数 // 每个step需要处理的任务量 (此处是整除)

总step数量 = num_train_epochs * 每个epoch包含的step数量

BTW:

  - step: 对应: 进行一次前向传播
  - epoch: 对应: 训练所有数据一次
