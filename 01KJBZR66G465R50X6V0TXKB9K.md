---
title: 20250131 - 使用ms-swift对Qwen2-VL进行微调 [22] - LR对推理能力的影响
confluence_page_id: 3344131
created_at: 2025-02-03T16:50:22+00:00
updated_at: 2025-02-07T02:27:09+00:00
---

# 背景

在 [20250131 - 使用ms-swift对Qwen2-VL进行微调 [21] - 训练一个识别基础原子的基模型] 中, 发现在课程学习中, 每个步骤的LR会逐步衰减 (使用了cos学习率优化器). 由于这种衰减是不可控的, 所以使用 --resume_only_model 让每步骤训练重新设定LR. 

需要对比每次重新设定LR, 和原来的方式, 哪一种的推理能力更好, 这种调整还有什么影响

# 使用原始方式, 让LR进行cos衰减

commit: 6cca2fa499b1dac646ff8f354b521a0af59945a8

脚本文件: run_tasks.random_selfies_only_basic_atom.sh

训练效果: 

```
# 步骤1
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158
 
评估纯原子数据, 长度9:
Correct: 35.00%, error_type
mismatch    65.0
            35.0

# 步骤2
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度9:
Correct: 40.00%, error_type
mismatch    60.0
            40.0

# 步骤3
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474
 
评估纯原子数据, 长度9:
Correct: 40.00%, error_type
mismatch    60.0
            40.0

# 步骤4
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474
 
评估纯原子数据, 长度9:
Correct: 40.00%, error_type
mismatch    60.0
            40.0

# 步骤5
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474
 
评估纯原子数据, 长度9:
Correct: 35.00%, error_type
mismatch    65.0
            35.0

# 步骤6
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474
 
评估纯原子数据, 长度9:
Correct: 45.00%, error_type
mismatch    55.0
            45.0
``` 

# 每次重新设定LR

commit: 31ab2b30cf4f4e3c721d9cdcdd9297568b52fb73

脚本文件: run_tasks.random_selfies_only_basic_atom.sh

训练效果: 

```
# 步骤1
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 52.63%, error_type
            52.631579
mismatch    47.368421
 
评估纯原子数据, 长度9:
Correct: 20.00%, error_type
mismatch    80.0
            20.0
 
# 步骤2
  
评估纯原子数据, 长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158
 
评估纯原子数据, 长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474
 
评估纯原子数据, 长度9:
Correct: 20.00%, error_type
mismatch    80.0
            20.0
``` 

可以看到, 每次重新设定LR, 会导致准确率下降

# 在每个阶段让LR减半

commit: 6e5d78bb0578b72c4ffe43249481386f0e5215a1

脚本文件: run_tasks.random_selfies_only_basic_atom.sh

训练效果: 

```
# 步骤1
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316
 
评估纯原子数据, 长度9:
Correct: 70.00%, error_type
            70.0
mismatch    30.0

# 步骤2
  
评估纯原子数据, 长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158
 
评估纯原子数据, 长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474
 
评估纯原子数据, 长度9:
Correct: 35.00%, error_type
mismatch    65.0
            35.0

# 步骤3
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474
 
评估纯原子数据, 长度9:
Correct: 40.00%, error_type
mismatch    60.0
            40.0

# 步骤4
  
评估纯原子数据, 长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158
 
评估纯原子数据, 长度8:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316
 
评估纯原子数据, 长度9:
Correct: 50.00%, error_type
            50.0
mismatch    50.0

# 步骤5
  
评估纯原子数据, 长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158
 
评估纯原子数据, 长度8:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632
 
评估纯原子数据, 长度9:
Correct: 35.00%, error_type
mismatch    65.0
            35.0

# 步骤6
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632
 
评估纯原子数据, 长度9:
Correct: 45.00%, error_type
mismatch    55.0
            45.0
``` 

其中: 步骤1的效果最好, 对已知数据和未知数据都有很好的效果.

而后面的步骤, 因为重启了LR, 导致效果下降厉害.

LR逐步下调, 有助于保持记忆和增强推理能力.

# 在每个步骤中段让LR减半, 在每个步骤也减半

commit: 4738f94438b3220259459096861c4679ff4a288a

脚本文件: run_tasks.random_selfies_only_basic_atom.sh

相当于: 

  - 步骤1:
    - 训练1: LR=4e-4
    - 训练2: LR=4e-4
    - 训练3: LR=2e-4
    - 训练4: LR=2e-4
  - 步骤2:
    - 训练1: LR=2e-4
    - 训练2: LR=2e-4
    - 训练3: LR=1e-4
    - 训练4: LR=1e-4
  - 步骤3:
    - 训练1: LR=1.3e-4
    - 训练2: LR=1.3e-4
    - 训练3: LR=0.65e-4
    - 训练4: LR=0.65e-4
  - 步骤4:
    - 训练1: LR=1e-4
    - 训练2: LR=1e-4
    - 训练3: LR=0.5e-4
    - 训练4: LR=0.5e-4

训练效果: 

```
# 步骤1
  
评估纯原子数据, 长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158
 
评估纯原子数据, 长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474
 
评估纯原子数据, 长度9:
Correct: 60.00%, error_type
            60.0
mismatch    40.0

# 步骤2
  
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0
 
评估纯原子数据, 长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474
 
评估纯原子数据, 长度9:
Correct: 45.00%, error_type
mismatch    55.0
            45.0

``` 

  
未见明显区别
