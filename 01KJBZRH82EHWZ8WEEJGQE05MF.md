---
title: 20250212 - 使用ms-swift对Qwen2-VL进行微调 [23] - 训练数据和lora层的关系
confluence_page_id: 3344294
created_at: 2025-02-12T05:38:38+00:00
updated_at: 2025-02-12T15:30:44+00:00
---

# 前置

[20250207 - 使用ms-swift对Qwen2-VL进行微调 [23] - 在基模型上增加原子数]

创建了一个lora权重变化比较的脚本, 查看不同的微调对lora层的影响. 

研究训练数据和lora层的关系

commit: f51aabfc00f3ad08dfb30066a04a2e1b46a9bfb7

# 已知基线

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 78.95% | 100.00% |
| 基础原子, 长度8 | 94.74% | 100.00% | 100.00% |
| ring分子, 长度7 | 97.50% | 100.00% | 97.50% |
| ring分子, 长度8 | 87.50% | 95.00% | 95.00% |
| branch分子, 长度7 | 83.33% | 90.00% | 93.33% |
| branch分子, 长度8 | 76.67% | 90.00% | 93.33% |
  
几个异常现象: 

  1. (基础原子, 长度7) 在步骤1/2是下降的, 在步骤3突然完美  

  2. (branch分子, 长度8) 通过增强数据是提升不大的, 但迭代步骤能提升
  3. (ring分子, 长度7) 的效果一致很好, 这部分是记忆保持能力

  

checkpoint:

  - 原始层: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v609-20250207-103318/checkpoint-318
  - 步骤1: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v817-20250212-002052/checkpoint-801
  - 步骤2: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v818-20250212-010602/checkpoint-801
  - 步骤3: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v819-20250212-015105/checkpoint-801

![image2025-2-12 11:8:40.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2011%3A8%3A40.png)

# 在步骤1上直接追加(基础原子, 长度7)

在步骤1上, 跑random_selfies_only_basic_atom, LR=1e-4, 1e-5, 1e-3

|  | 基线 (epoch=3) (LR=1e-4) | 追加 (epoch=6) (LR=1e-4) | 追加 (epoch=6) (LR=1e-5) | 追加 (epoch=6) (LR=1e-3) |
| --- | --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 100.00% | 100.00% | 84.21% |
| 基础原子, 长度8 | 94.74% | 100.00% | 94.74% | 52.63% |
| ring分子, 长度7 | 97.50% | 92.50% | 97.50% | 0.00% |
| ring分子, 长度8 | 87.50% | 92.50% | 87.50% | 0.00% |
| branch分子, 长度7 | 83.33% | 90.00% | 86.67% | 43.33% |
| branch分子, 长度8 | 76.67% | 70.00% | 80.00% | 36.67% |
|  | ![image2025-2-12 11:25:10.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2011%3A25%3A10.png) | ![image2025-2-12 11:34:30.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2011%3A34%3A30.png) | ![image2025-2-12 13:9:27.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2013%3A9%3A27.png) |  |
  
# 在步骤1上直接追加(branch_selfies)数据

在步骤1上, 跑branch_selfies, LR=1e-4 或 LR=1e-5

|  | 基线 (epoch=3) (LR=1e-4) | 追加 (epoch=6) (LR=1e-4) | 追加 (epoch=6) (LR=1e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 52.63% | 63.16% |
| 基础原子, 长度8 | 94.74% | 26.32% | 68.42% |
| ring分子, 长度7 | 97.50% | 97.50% | 97.50% |
| ring分子, 长度8 | 87.50% | 85.00% | 87.50% |
| branch分子, 长度7 | 83.33% | 80.00% | 83.33% |
| branch分子, 长度8 | 76.67% | 63.33% | 80.00% |
|  | ![image2025-2-12 13:15:3.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2013%3A15%3A3.png) | ![image2025-2-12 14:13:20.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2014%3A13%3A20.png) |  |
  
  

  

追加(branch_selfies), 影响最大的是 (基础院子, 长度8), 认为其对应layer 22-24, 

  

# 在步骤1上直接追加(rotate_selfies_substring_from_branch_selfies*2)数据

在步骤1上, 跑rotate_selfies_substring_from_branch_selfies*2, LR=1e-4 或 LR=1e-5

|  | 基线 (epoch=3) (LR=1e-4) | 追加 (epoch=6) (LR=1e-4) | 追加 (epoch=6) (LR=1e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 84.21% | 100.00% |
| 基础原子, 长度8 | 94.74% | 84.21% | 100.00% |
| ring分子, 长度7 | 97.50% | 87.50% | 95.00% |
| ring分子, 长度8 | 87.50% | 85.00% | 87.50% |
| branch分子, 长度7 | 83.33% | 90.00% | 86.67% |
| branch分子, 长度8 | 76.67% | 83.33% | 80.00% |
|  | ![image2025-2-12 13:23:44.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2013%3A23%3A44.png) | ![image2025-2-12 14:15:53.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2014%3A15%3A53.png) |  |
  
  

  

这里热力图上跟上一步的猜测(" (基础院子, 长度8), 认为其对应layer 22-24") 冲突

# 在步骤1上直接追加(add_branch_op_from_random_selfies_only_basic_atom*3)数据

在步骤1上, 跑add_branch_op_from_random_selfies_only_basic_atom*3, LR=1e-4 或 LR=1e-5

|  | 基线 (epoch=3) (LR=1e-4) | 追加 (epoch=6) (LR=1e-4) | 追加 (epoch=6) (LR=1e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 47.37% | 63.16% |
| 基础原子, 长度8 | 94.74% | 52.63% | 94.74% |
| ring分子, 长度7 | 97.50% | 95.00% | 97.50% |
| ring分子, 长度8 | 87.50% | 85.00% | 90.00% |
| branch分子, 长度7 | 83.33% | 73.33% | 93.33% |
| branch分子, 长度8 | 76.67% | 80.00% | 73.33% |
|  | ![image2025-2-12 13:31:20.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2013%3A31%3A20.png) | ![image2025-2-12 14:31:1.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2014%3A31%3A1.png) |  |
  
  

  

# 在步骤1上直接追加(branch_selfies_by_diff_start_point_from_random_selfies_only_basic_atom*2)数据

在步骤1上, branch_selfies_by_diff_start_point_from_random_selfies_only_basic_atom*2, LR=1e-4 或 LR=1e-5

|  | 基线 (epoch=3) (LR=1e-4) | 追加 (epoch=6) (LR=1e-4) | 追加 (epoch=6) (LR=1e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 89.47% | 78.95% |
| 基础原子, 长度8 | 94.74% | 84.21% | 84.21% |
| ring分子, 长度7 | 97.50% | 97.50% | 97.50% |
| ring分子, 长度8 | 87.50% | 82.50% | 80.00% |
| branch分子, 长度7 | 83.33% | 90.00% | 90.00% |
| branch分子, 长度8 | 76.67% | 73.33% | 76.67% |
|  | ![image2025-2-12 13:34:37.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2013%3A34%3A37.png) | ![image2025-2-12 14:32:50.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2014%3A32%3A50.png) |  |
  
  

# 在步骤1上直接追加(branch_selfies_by_diff_start_point_from_rotate_selfies_substring_from_random_selfies_only_basic_atom*2)数据, LR=1e-4

在步骤1上, branch_selfies_by_diff_start_point_from_rotate_selfies_substring_from_random_selfies_only_basic_atom*2, LR=1e-4

|  | 基线 (epoch=3) (LR=1e-4) | 追加 (epoch=6) (LR=1e-4) | 追加 (epoch=6) (LR=1e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 21.05% | 47.37% |
| 基础原子, 长度8 | 94.74% | 5.26% | 26.32% |
| ring分子, 长度7 | 97.50% | 87.50% | 100.00% |
| ring分子, 长度8 | 87.50% | 52.50% | 82.50% |
| branch分子, 长度7 | 83.33% | 36.67% | 80.00% |
| branch分子, 长度8 | 76.67% | 3.33% | 60.00% |
|  | ![image2025-2-12 13:36:48.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2013%3A36%3A48.png) | ![image2025-2-12 14:36:41.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2014%3A36%3A41.png) |  |
  
  

# 在步骤1上直接追加(rotate_selfies_substring_from_branch_selfies*2)数据, 使用不同的epoch

|  | 基线 (epoch=3) (LR=1e-4) | 追加 (epoch=1) (LR=1e-5) | 追加 (epoch=2) (LR=1e-5) | 追加 (epoch=3) (LR=1e-5) | 追加 (epoch=4) (LR=1e-5) |
| --- | --- | --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 94.74% | 89.47% | 94.74% | 89.47% |
| 基础原子, 长度8 | 94.74% | 100.00% | 94.74% | 100.00% | 89.47% |
| ring分子, 长度7 | 97.50% | 97.50% | 97.50% | 100.00% | 97.50% |
| ring分子, 长度8 | 87.50% | 90.00% | 92.50% | 85.00% | 82.50% |
| branch分子, 长度7 | 83.33% | 83.33% | 83.33% | 76.67% | 76.67% |
| branch分子, 长度8 | 76.67% | 80.00% | 76.67% | 83.33% | 80.00% |
|  | 数据量太小, 对各层的改变比较均匀, |  |  |  |  |
| 热力图没有意义 | ![image2025-2-12 15:28:59.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2015%3A28%3A59.png) | ![image2025-2-12 15:34:13.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2015%3A34%3A13.png) | ![image2025-2-12 15:48:52.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2015%3A48%3A52.png) |  |  |
|  | ![image2025-2-12 15:21:40.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2015%3A21%3A40.png) | ![image2025-2-12 15:29:4.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2015%3A29%3A4.png) | ![image2025-2-12 15:34:8.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2015%3A34%3A8.png) | ![image2025-2-12 15:49:5.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2015%3A49%3A5.png) |  |
|  |  |  |  |  |  |
|  | 基线 (epoch=3) (LR=1e-4) | 追加 (epoch=5) (LR=1e-5) | 追加 (epoch=6) (LR=1e-5) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 89.47% | 94.74% |
| 基础原子, 长度8 | 94.74% | 100.00% | 100.00% |
| ring分子, 长度7 | 97.50% | 97.50% | 95.00% |
| ring分子, 长度8 | 87.50% | 80.00% | 82.50% |
| branch分子, 长度7 | 83.33% | 83.33% | 80.00% |
| branch分子, 长度8 | 76.67% | 76.67% | 80.00% |
|  | ![image2025-2-12 16:3:33.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A3%3A33.png) | ![image2025-2-12 16:20:33.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A20%3A33.png) |  |
|  | ![image2025-2-12 16:3:41.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A3%3A41.png) | ![image2025-2-12 16:20:44.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A20%3A44.png) |  |
|  | 基线 (epoch=3) (LR=1e-4) | 追加 (epoch=1) (LR=1e-4) | 追加 (epoch=2) (LR=1e-4) | 追加 (epoch=3) (LR=1e-4) |
| --- | --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 94.74% | 94.74% | 89.47% |
| 基础原子, 长度8 | 94.74% | 100.00% | 94.74% | 89.47% |
| ring分子, 长度7 | 97.50% | 95.00% | 97.50% | 100.00% |
| ring分子, 长度8 | 87.50% | 85.00% | 82.50% | 87.50% |
| branch分子, 长度7 | 83.33% | 90.00% | 90.00% | 83.33% |
| branch分子, 长度8 | 76.67% | 90.00% | 76.67% | 80.00% |
|  | ![image2025-2-12 16:27:53.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A27%3A53.png) | ![image2025-2-12 16:54:30.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A54%3A30.png) | ![image2025-2-12 16:56:3.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A56%3A3.png) |  |
|  | ![image2025-2-12 16:28:6.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A28%3A6.png) | ![image2025-2-12 16:54:38.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A54%3A38.png) | ![image2025-2-12 16:56:18.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A56%3A18.png) |  |
|  | 基线 (epoch=3) (LR=1e-4) | 追加 (epoch=4) (LR=1e-4) | 追加 (epoch=5) (LR=1e-4) | 追加 (epoch=6) (LR=1e-4) |
| --- | --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 89.47% | 84.21% | 68.42% |
| 基础原子, 长度8 | 94.74% | 100.00% | 89.47% | 94.74% |
| ring分子, 长度7 | 97.50% | 97.50% | 100.00% | 100.00% |
| ring分子, 长度8 | 87.50% | 82.50% | 85.00% | 82.50% |
| branch分子, 长度7 | 83.33% | 73.33% | 80.00% | 76.67% |
| branch分子, 长度8 | 76.67% | 83.33% | 70.00% | 73.33% |
|  | ![image2025-2-12 16:58:36.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A58%3A36.png) | ![image2025-2-12 16:59:50.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A59%3A50.png) | ![image2025-2-12 17:42:8.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2017%3A42%3A8.png) |  |
|  | ![image2025-2-12 16:58:44.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A58%3A44.png) | ![image2025-2-12 16:59:57.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2016%3A59%3A57.png) | ![image2025-2-12 17:42:17.png](/assets/01KJBZRH82EHWZ8WEEJGQE05MF/image2025-2-12%2017%3A42%3A17.png) |  |
  
  

epoch=1时, 对于branch, 出现过一次90%准确率, 但复现时无法复现.

怀疑是评估问题, 尝试多次运行评估, 查看其偏差

发现评估脚本的temprature是0.6, 会导致准确率的突然异常. 修改脚本将temperature设置为0.01, 进行10次评估, 评估偏差在10%以内

从热力图可以看出: 

  - LR影响到每一层调整的幅度
  - 随着epoch进行, 每一层调整的幅度逐步升高
  - 随着epoch进行: 在初期, 每一层调整幅度差不都. 随着epoch进行, 各层差异就有差异, 并且重点逐步集中
