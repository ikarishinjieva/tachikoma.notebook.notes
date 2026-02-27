---
title: 20250207 - 使用ms-swift对Qwen2-VL进行微调 [23] - 在基模型上增加原子数
confluence_page_id: 3344202
created_at: 2025-02-07T08:59:15+00:00
updated_at: 2025-02-12T05:02:46+00:00
---

# 前继

[20250131 - 使用ms-swift对Qwen2-VL进行微调 [21] - 训练一个识别基础原子的基模型]

目前的checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v609-20250207-103318/checkpoint-318 , 是训练方式30的步骤一, 其效果: 

|  | 掺入等量的不同起点的branch数据 |
| --- | --- |
      
    
    步骤=1, 评估普通分子, 长度7

| 94.74%  
      
    
    步骤=1, 评估普通分子, 长度8

| 68.42%  
      
    
    步骤=1, 评估ring分子, 长度7

| 87.50%  
      
    
    步骤=1, 评估ring分子, 长度8

| 27.50%  
      
    
    步骤=1, 评估branch分子, 长度7

| 90.00%  
      
    
    步骤=1, 评估branch分子, 长度8

| 80.00%  
  
将目前的训练步骤分步骤放在脚本文件中: commit: e141d644434ec5d4c0b29cfea78e23d1feed2abe

  - run_tasks.01.random_selfies_only_basic_atom.sh
  - run_tasks.02.ring_selfies.sh
  - run_tasks.03.branch_selfies.sh

整理目前的训练步骤: 

  1. 训练random_selfies_only_basic_atom (训练方式11)
     1. 随机生成的分子:

        - 长度3(50个)+长度4 (100个), 带等量的镜像增强数据, 带4倍的子段轮换的增强数据-v4
        - 长度3(25个)+长度4(50个)+长度5 (100个), 带等量的镜像增强数据, 带4倍的子段轮换的增强数据-v4
        - 长度3(12个)+长度4(25个)+长度5(50个)+长度6 (100个), 带等量的镜像增强数据, 带4倍的子段轮换的增强数据-v4
        - 长度3(6个)+长度4(12个)+长度5(25个)+长度6(50个)+长度7 (100个), 带等量的镜像增强数据, 带4倍的子段轮换的增强数据-v4
  2. 训练ring_selfies (训练方式21)
     1. 训练ring, 主要步骤:

        - ring_selfies: 长度5(35个), 长度6(70个), 掺入等量的部分普通分子, 3倍的基于ring分子的rotate训练数据
        - ring_selfies: 长度5(17个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 3倍的基于ring分子的rotate训练数据
  3. 训练branch_selfies (训练方式30)
     1. 训练branch, 掺入 不同起点的branch数据, 主要步骤:

        - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个), 掺入等量的部分普通分子, 掺入等量的ring分子, 掺入2倍的branch分子的rotate增强, LR=4e-4, 掺入等量的branch增强数据(添加[branch]操作符), 掺入等量的不同起点的branch数据
        - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 掺入等量的ring分子, 掺入2倍的branch分子的rotate增强, LR=2e-4, 掺入等量的branch增强数据(添加[branch]操作符), 掺入等量的不同起点的branch数据

下一步: 在其上 提高长度8/9 的准确率

# 训练1

在之前习得了 基础院子+ring+branch的模型checkpoint基础上, 进行如下训练: 

  - 阶段1: 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4

  - 阶段2: 长度5-8的ring原子, 长度5-8的原子, 长度3-8的基础原子

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:2
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:2
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_4.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_3.train.json:2
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:2
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_4.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_3.train.json:2
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:2
 
---
 
 
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:2
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:2
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_8.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:8

``` 

训练效果: 

|  | 步骤1 (epoch=3) | 步骤2 (epoch=6) | 步骤3 (epoch=9) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 78.95% | 89.47% | 94.74% |
| 基础原子, 长度8 | 78.95% | 94.74% | 89.47% |
| ring分子, 长度7 | 80.00% | 85.00% | 90.00% |
| ring分子, 长度8 | 75.00% | 67.50% | 67.50% |
| branch分子, 长度7 | 60.00% | 73.33% | 63.33% |
| branch分子, 长度8 | 40.00% | 46.67% | 53.33% |
  
分析: 

  - branch分子的识别率, 和长度8的ring分子的识别率都不够, 尝试增加相关数据

# 训练2

在之前习得了 基础院子+ring+branch的模型checkpoint基础上, 进行如下训练: 

  - 阶段1: 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4

  - 阶段2: 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 带1/2数量的长度3-8的基础原子(保持记忆)

  - 阶段3: 长度5-8的branch原子, 带2倍的基于branch分子的rotate训练数据, 带1/2数量的长度3-8的基础原子(保持记忆), 带1/2数量的长度5-8的ring原子(保持记忆)

训练日志: [run_tasks.2310.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/run_tasks.2310.log)

训练效果:

|  | 步骤1 (epoch=3) (LR=4e-4) | 步骤2 (epoch=6) (LR=2e-4) | 步骤3 (epoch=9) (LR=1e-4) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 68.42% | 63.16% | 89.47% |
| 基础原子, 长度8 | 84.21% | 68.42% | 89.47% |
| ring分子, 长度7 | 90.00% | 95.00% | 90.00% |
| ring分子, 长度8 | 65.00% | 90.00% | 87.50% |
| branch分子, 长度7 | 66.67% | 70.00% | 76.67% |
| branch分子, 长度8 | 73.33% | 70.00% | 93.33% |
  
分析: 

  - ring的记忆保持是最好的
  - 在步骤1: 基础原子 和 branch分子 都出现了 学习>记忆 的情况 (考虑调低初始LR)
  - 在步骤3: 基础原子和ring分子都达标, 但branch分子仍然是 学习>记忆 的情况 (但长度7的branch分子的训练数据量并不小)

重跑一次:

|  | 步骤1 (epoch=3) (LR=4e-4) | 步骤2 (epoch=6) (LR=2e-4) | 步骤3 (epoch=9) (LR=1e-4) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 63.16% | 84.21% | 68.42% |
| 基础原子, 长度8 | 73.68% | 89.47% | 89.47% |
| ring分子, 长度7 | 82.50% | 80.00% | 82.50% |
| ring分子, 长度8 | 60.00% | 67.50% | 80.00% |
| branch分子, 长度7 | 76.67% | 90.00% | 96.67% |
| branch分子, 长度8 | 60.00% | 76.67% | 76.67% |
  
结论有变化: 

  - 记忆保持最好的是branch分子
  - 在步骤3: 基础原子出现了遗忘, 长度8的branch分子效果仍然不好, ring分子的效果也不好

在相同参数的两次运行中出现了随机性, 需要考虑如何将其稳定下来

先尝试调低整体LR, 来保持记忆力

# 训练3

同训练2, 调低整体LR: 

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 78.95% | 84.21% |
| 基础原子, 长度8 | 94.74% | 100.00% | 100.00% |
| ring分子, 长度7 | 80.00% | 92.50% | 90.00% |
| ring分子, 长度8 | 82.50% | 77.50% | 77.50% |
| branch分子, 长度7 | 76.67% | 83.33% | 93.33% |
| branch分子, 长度8 | 66.67% | 73.33% | 70.00% |
  
降低整体LR的情况下, 比上一轮效果要好一点 (整体平衡性提升)

增加训练步骤:

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) | 步骤4 (epoch=12) (LR=3e-05) | 步骤5 (epoch=15) (LR=2e-05) |
| --- | --- | --- | --- | --- | --- |
| 基础原子, 长度7 | 78.95% | 89.47% | 84.21% | 78.95% | 89.47% |
| 基础原子, 长度8 | 84.21% | 100.00% | 100.00% | 94.74% | 94.74% |
| ring分子, 长度7 | 85.00% | 85.00% | 87.50% | 87.50% | 87.50% |
| ring分子, 长度8 | 85.00% | 75.00% | 75.00% | 82.50% | 77.50% |
| branch分子, 长度7 | 76.67% | 86.67% | 90.00% | 83.33% | 83.33% |
| branch分子, 长度8 | 70.00% | 70.00% | 80.00% | 63.33% | 66.67% |
  
随着步骤增加, 效果并不增加.

查看 基础原子, 长度7 的 "记忆遗忘" 的错误结果(标红) :

```
5   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/75.png    {
"selfies": "[O][Branch1][Ring1][N][O][C][C][O][N]"}        O(NO)CCON  {
"selfies": "[N][C][O][O][N][O][C]"}   NCOONOC    0.137931   
复杂变换: [N][O][C][C][O][N][O] 和 [N][C][O][O][N][O][C] 的差异

13  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/79.png  {
"selfies": "[C][Branch1][Branch1][C][N][C][C][O][N]"}        C(CNCC)ON  {
"selfies": "[O][C][C][N][C][C][N]"}   OCCNCCN    0.185185   
镜像([N][O][C][C][N][C][C]), 1-7轮换

17  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/95.png    {
"selfies": "[O][Branch1][Ring1][N][O][N][C][O][C]"}        O(NO)NCOC  {
"selfies": "[O][C][N][O][O][N][C]"}   OCNOONC    0.222222   
复杂变换: [O][N][O][N][C][O][C] 和 [O][C][N][O][O][N][C]的差异

16   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/4.png                    {
"selfies": "[C][N][C][O][C][O][C]"}          CNCOCOC  {
"selfies": "[O][C][C][O][C][N][C]"}   OCCOCNC    0.434783   
镜像, 6-7轮换
 
---
 
9   /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/56.png    {
"selfies": "[O][Branch1][Ring1][N][N][N][C][C][O]"}        O(NN)NCCO  {
"selfies": "[C][N][N][N][O][C][O]"}   CNNNOCO    0.172414   
mismatch
复杂变化: [O][C][C][N][O][N][N]和[O][C][O][N][N][N][C]的差异

13  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/79.png  {
"selfies": "[C][Branch1][Branch1][C][N][C][C][O][N]"}        C(CNCC)ON  {
"selfies": "[O][C][C][N][C][C][N]"}   OCCNCCN    0.185185   
mismatch
复杂变化: [N][O][C][C][N][C][C]和[N][C][C][N][C][C][O]的差异

14  /opt/huangyan/mol-ai/images/random_selfies_only_basic_atom/length_7/78.png    {
"selfies": "[O][Branch1][Ring1][O][N][C][O][C][N]"}        O(ON)COCN  {
"selfies": "[N][C][O][O][O][C][N]"}   NCOOOCN    0.315789   
mismatch
起点2([N][O][O][C][O][C][N]), 5-6轮换

``` 

看起来不是"记忆遗忘", 而是branch结构对于传统链式分子产生了影响

查看 branch原子, 长度8 的 错误结果(标绿) :

```
 29   /opt/huangyan/mol-ai/images/branch_selfies/length_8/37.png    {
"selfies": "[N][Branch1][Ring2][C][N][C][O][O]"}         N(CNC)OO    {
"selfies": "[C][Branch1][Ring1][O][N][N][C][O]"}  C(ON)NCO    0.160000   
mismatch
差异: [O][C][N][C][O][N] 和 [O][O][N][C][N][C]

12   /opt/huangyan/mol-ai/images/branch_selfies/length_8/75.png    {
"selfies": "[O][Branch1][Ring2][N][C][C][O][C]"}         O(NCC)OC    {
"selfies": "[N][Branch1][Ring2][C][O][C][C][O]"}  N(COC)CO    0.208333   
mismatch
差异: [C][O][C][N][C][O] 和 [C][O][O][N][C][C]

10   /opt/huangyan/mol-ai/images/branch_selfies/length_8/64.png        {
"selfies": "[O][O][N][Branch1][C][O][N][C]"}         OON(O)NC    {
"selfies": "[N][O][N][Branch1][Ring1][O][O][C]"}  NON(OO)C    0.272727   
mismatch
起点4: [O][O][N][Branch1][C][C][O][N], 分支内外轮换

9    /opt/huangyan/mol-ai/images/branch_selfies/length_8/38.png    {
"selfies": "[N][Branch1][Ring1][O][O][N][N][C]"}         N(OO)NNC    {
"selfies": "[O][Branch1][Ring1][O][N][N][N][C]"}  O(ON)NNC    0.272727   
mismatch
差异: [C][N][N][N][O][O]和[C][N][N][O][O][N]

28  /opt/huangyan/mol-ai/images/branch_selfies/length_8/105.png    {
"selfies": "[N][Branch1][Ring2][O][O][C][N][C]"}         N(OOC)NC    {
"selfies": "[O][Branch1][Ring2][N][C][N][O][C]"}  O(NCN)OC    0.333333   
mismatch
差异: [C][O][O][N][N][C]和[C][O][O][N][C][N]

8    /opt/huangyan/mol-ai/images/branch_selfies/length_8/32.png    {
"selfies": "[O][C][Branch1][Ring1][N][O][O][N]"}         OC(NO)ON        {
"selfies": "[O][N][C][Branch1][C][N][O][O]"}  ONC(N)OO    0.333333   
mismatch
起点4: [N][C][Branch1][Ring1][N][O][O][O], 1-7对调

``` 

发现长度7 的基础原子的"记忆遗忘", 和branch的错误, 都跟branch的增强数据要解决的问题相关.

先加回增强数据: 

  - branch增强数据(添加[branch]操作符)
  - 不同起点的branch数据

# 训练4

加入branch的两类增强数据, 进行如下训练: 

  - 阶段1: 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4

  - 阶段2: 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 带1/2数量的长度3-8的基础原子(保持记忆)

  - 阶段3: 长度5-8的branch原子, 带2倍的基于branch分子的rotate训练数据, 掺入等量的branch增强数据(添加[branch]操作符), 掺入等量的不同起点的branch数据, 带1/2数量的长度3-8的基础原子(保持记忆), 带1/2数量的长度5-8的ring原子(保持记忆)

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 89.47% | 89.47% | 68.42% |
| 基础原子, 长度8 | 78.95% | 84.21% | 78.95% |
| ring分子, 长度7 | 87.50% | 95.00% | 95.00% |
| ring分子, 长度8 | 77.50% | 80.00% | 80.00% |
| branch分子, 长度7 | 83.33% | 90.00% | 96.67% |
| branch分子, 长度8 | 70.00% | 93.33% | 93.33% |
  
branch的识别效果稳定下来. 发现: 

  1. 基础原子在步骤3出现衰减, 衰减原因是 更多的分子被识别成带branch的结构
  2. 长度8的ring分子识别率停在80%, 分析其错误: [length_8.ring.analyzed.1932.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/length_8.ring.analyzed.1932.log) : 
     1. 错误分布: 

```
将ring结构识别成了branch结构, 环内和分支内的原子相同
将ring结构识别成了branch结构, 环内和分支内的原子相同

ring操作符应往后移一位
ring操作符应往后移一位
ring操作符应往后移一位, 然后5-6轮换
镜像, ring操作符应往后移一位, 5和8对调

5-6轮换
1-3轮换
``` 

数据增强方向: 

  1. 将普通分子的起点改变, 使其形成branch结构, 让模型理解两者差异
  2. 模仿add_branch_op_from_random_selfies_only_basic_atom, 在基础分子中几处增加ring操作符, 让模型理解ring操作符的位置

# 训练5

加入 改变普通分子起点形成的branch分子(注意长度是n+2), 替换"不同起点的branch数据", 进行如下训练: 

  - 阶段1: 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4

  - 阶段2: 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 带1/2数量的长度3-8的基础原子(保持记忆)

  - 阶段3: 长度5-8的branch原子, 带2倍的基于branch分子的rotate训练数据, 掺入等量的branch增强数据(添加[branch]操作符), 掺入等量的改变普通分子起点形成的branch分子 (注意长度是5-10), 带1/2数量的长度3-8的基础原子(保持记忆), 带1/2数量的长度5-8的ring原子(保持记忆)

训练日志: [run_tasks.0118.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/run_tasks.0118.log)

掺入等量的改变普通分子起点形成的branch分子:

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 84.21% | 94.74% | 89.47% |
| 基础原子, 长度8 | 63.16% | 89.47% | 94.74% |
| ring分子, 长度7 | 92.50% | 95.00% | 95.00% |
| ring分子, 长度8 | 77.50% | 80.00% | 85.00% |
| branch分子, 长度7 | 63.33% | 83.33% | 100.00% |
| branch分子, 长度8 | 73.33% | 83.33% | 93.33% |
  
检查基础原子的评估输出, 大部分原子都带有branch结构, 也就是说增强数据是生效的.

从效果上, 基础原子的长度7的效果得以保持.

再检查ring分子的问题 (标红部分): 

  - 4个ring op往后移, 移到最后一位或者倒数第二位
  - 1个ring 长度错误
  - 1个普通轮换

与训练4的结论一致, 可以增加一些ring的增强数据

掺入2倍的改变普通分子起点形成的branch分子:

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 78.95% | 94.74% | 100.00% |
| 基础原子, 长度8 | 78.95% | 94.74% | 100.00% |
| ring分子, 长度7 | 92.50% | 95.00% | 90.00% |
| ring分子, 长度8 | 82.50% | 77.50% | 77.50% |
| branch分子, 长度7 | 76.67% | 96.67% | 93.33% |
| branch分子, 长度8 | 63.33% | 86.67% | 83.33% |
  
  

对于普通分子的正确率上升

# 训练5-额外

去掉 "掺入等量的branch增强数据(添加[branch]操作符)" 增强数据, 进行如下训练: 

  - 阶段1: 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4

  - 阶段2: 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 带1/2数量的长度3-8的基础原子(保持记忆)

  - 阶段3: 长度5-8的branch原子, 带2倍的基于branch分子的rotate训练数据, ~~掺入等量的branch增强数据(添加[branch]操作符),~~ 掺入等量的改变普通分子起点形成的branch分子 (注意长度是5-10), 带1/2数量的长度3-8的基础原子(保持记忆), 带1/2数量的长度5-8的ring原子(保持记忆)

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 63.16% | 84.21% | 89.47% |
| 基础原子, 长度8 | 73.68% | 78.95% | 94.74% |
| ring分子, 长度7 | 95.00% | 95.00% | 90.00% |
| ring分子, 长度8 | 77.50% | 72.50% | 70.00% |
| branch分子, 长度7 | 83.33% | 93.33% | 83.33% |
| branch分子, 长度8 | 63.33% | 70.00% | 80.00% |
  
  

基础原子和branch分子的效果都在下降

# 训练6

加入 等量的ring增强数据(添加[ring]操作符), 进行如下训练: 

  - 阶段1: 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4

  - 阶段2: 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 掺入等量的ring增强数据(添加[ring]操作符), 带1/2数量的长度3-8的基础原子(保持记忆)

  - 阶段3: 长度5-8的branch原子, 带2倍的基于branch分子的rotate训练数据, 掺入等量的branch增强数据(添加[branch]操作符), 掺入2倍的改变普通分子起点形成的branch分子 (注意长度是5-10), 带1/2数量的长度3-8的基础原子(保持记忆), 带1/2数量的长度5-8的ring原子(保持记忆)

commit: 07375439df4b26dc157d436c94f505487574472d

训练日志: [run_tasks.2133.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/run_tasks.2133.log)

训练效果:

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 78.95% | 94.74% | 89.47% |
| 基础原子, 长度8 | 84.21% | 94.74% | 94.74% |
| ring分子, 长度7 | 85.00% | 90.00% | 92.50% |
| ring分子, 长度8 | 67.50% | 80.00% | 75.00% |
| branch分子, 长度7 | 93.33% | 86.67% | 90.00% |
| branch分子, 长度8 | 63.33% | 76.67% | 80.00% |
  
分析ring错误: [length_8.ring.analyzed.2208.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/length_8.ring.analyzed.2208.log)

掺入2倍的ring增强数据(添加[ring]操作符):

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=9) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 78.95% | 73.68% | 94.74% |
| 基础原子, 长度8 | 68.42% | 89.47% | 94.74% |
| ring分子, 长度7 | 97.50% | 97.50% | 97.50% |
| ring分子, 长度8 | 62.50% | 77.50% | 77.50% |
| branch分子, 长度7 | 80.00% | 86.67% | 93.33% |
| branch分子, 长度8 | 73.33% | 83.33% | 73.33% |
  
  

(ring分子, 长度8)并未提升, 查看错误分布:

  - [length_8.ring.analyzed.1.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/length_8.ring.analyzed.1.log)
  - [length_8.ring.analyzed.2.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/length_8.ring.analyzed.2.log)

  

"等量的ring增强数据(添加[ring]操作符)" 并没有改善模型对ring位置的认知. 需要其他方式的增强: 

  - 将某一段ring结构, 整体平移
  - 对环内的原子进行轮换

  

# 训练7

加入 等量的ring平移数据, 进行如下训练: 

  - 阶段1: 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4

  - 阶段2: 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 等量的ring平移数据, 带1/2数量的长度3-8的基础原子(保持记忆)

  - 阶段3: 长度5-8的branch原子, 带2倍的基于branch分子的rotate训练数据, 掺入等量的branch增强数据(添加[branch]操作符), 掺入2倍的改变普通分子起点形成的branch分子 (注意长度是5-10), 带1/2数量的长度3-8的基础原子(保持记忆), 带1/2数量的长度5-8的ring原子(保持记忆)

commit: 833df8a21aef0195ec6160d47bab51b312a4c902

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) |
| --- | --- | --- |
| 基础原子, 长度7 | 78.95% | 89.47% |
| 基础原子, 长度8 | 94.74% | 100.00% |
| ring分子, 长度7 | 92.50% | 92.50% |
| ring分子, 长度8 | 75.00% | 67.50% |
| branch分子, 长度7 | 90.00% | 83.33% |
| branch分子, 长度8 | 80.00% | 90.00% |
  
长度8的ring分子识别率并没有改善. 分析其错误分布: 一共13个错误, 2个缺原子, 5个环右移, 4个环内原子交换, 2个ring识别成了branch

增大ring平移数据的数量, 使用2倍的ring平移数据: 

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤2 (epoch=6) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 89.47% | 89.47% | 100.00% |
| 基础原子, 长度8 | 78.95% | 89.47% | 94.74% |
| ring分子, 长度7 | 87.50% | 92.50% | 90.00% |
| ring分子, 长度8 | 67.50% | 70.00% | 70.00% |
| branch分子, 长度7 | 80.00% | 83.33% | 90.00% |
| branch分子, 长度8 | 73.33% | 86.67% | 90.00% |
  
增大ring平移数据的数量, 使用3倍的ring平移数据: 

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤2 (epoch=6) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 78.95% | 94.74% | 100.00% |
| 基础原子, 长度8 | 68.42% | 94.74% | 94.74% |
| ring分子, 长度7 | 95.00% | 95.00% | 97.50% |
| ring分子, 长度8 | 62.50% | 70.00% | 77.50% |
| branch分子, 长度7 | 80.00% | 90.00% | 83.33% |
| branch分子, 长度8 | 76.67% | 83.33% | 80.00% |
  
  

ring分子, 长度8 的效果出现下降.

准备单独训练ring分子, 查看结果. 先评估以上训练中, 每个(刚训练完ring分子的步骤)的checkpoint

  - 2倍的ring平移数据, 步骤1: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v739-20250210-014718/checkpoint-201
  - 2倍的ring平移数据, 步骤2: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v742-20250210-022702/checkpoint-201

  - 2倍的ring平移数据, 步骤3: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v745-20250210-030700/checkpoint-201

  - 3倍的ring平移数据, 步骤1: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v748-20250210-034656/checkpoint-222

  - 3倍的ring平移数据, 步骤2: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v751-20250210-042752/checkpoint-222
  - 3倍的ring平移数据, 步骤3: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v754-20250210-050848/checkpoint-222

  

2倍的ring平移数据, 训练效果: 

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤2 (epoch=6) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| ring分子, 长度7 | 90.00% | 95.00% | 92.50% |
| ring分子, 长度8 | 77.50% | 77.50% | 77.50% |
| branch分子, 长度7 | 93.33% | 96.67% | 100.00% |
| branch分子, 长度8 | 60.00% | 83.33% | 96.67% |
  
3倍的ring平移数据, 训练效果: 

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤2 (epoch=6) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 94.74% | 100.00% |
| 基础原子, 长度8 | 100.00% | 94.74% | 89.47% |
| ring分子, 长度7 | 92.50% | 100.00% | 97.50% |
| ring分子, 长度8 | 77.50% | 72.50% | 77.50% |
| branch分子, 长度7 | 93.33% | 90.00% | 96.67% |
| branch分子, 长度8 | 60.00% | 70.00% | 96.67% |
  
  

结论: 

  1. 最后一个步骤 (训练branch的步骤), 会带来整体效果的下降
  2. 刚训练完Ring之后, 评估Ring的正确率也只有77.50%, 认为数据增强方向有问题

# 训练8

仅训练ring数据, 进行如下训练: 

  - 阶段1: 

    - 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4; 

    - 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 2倍的ring平移数据

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤2 (epoch=6) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| ring分子, 长度7 | 95.00% | 97.50% | 100.00% |
| ring分子, 长度8 | 77.50% | 82.50% | 77.50% |
| branch分子, 长度7 | 86.67% | 93.33% | 93.33% |
| branch分子, 长度8 | 60.00% | 66.67% | 63.33% |
  

分析错误分布: [length_8.ring.analyzed.1307.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/length_8.ring.analyzed.1307.log)

错误5/7出现在 ring内的原子轮换

  

# 训练9

增加在ring环内进行整段轮换的数据, 进行如下训练: 

  - 阶段1: 

    - 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4; 

    - 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 2倍的ring平移数据, 等量的在ring环内进行整段轮换的数据

  

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) |
| --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% |
| 基础原子, 长度8 | 94.74% | 94.74% |
| ring分子, 长度7 | 95.00% | 97.50% |
| ring分子, 长度8 | 90.00% | 95.00% |
| branch分子, 长度7 | 86.67% | 93.33% |
| branch分子, 长度8 | 46.67% | 40.00% |
  
  

commit: f4e98380fff81ef9f6656f6c998879da5cfef530

训练日志: [run_tasks.1414.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/run_tasks.1414.log)

训练效果很好. 分析标红部分的错误分布: 

```
 25  /opt/huangyan/mol-ai/images/ring_selfies/length_8/170.png   {
"selfies": "[C][O][N][N][N][N][Ring1][Branch1]"}           CONNNN   {
"selfies": "[N][O][N][N][N][Ring1][Branch1][C]"}  N1ONNN1C    0.035714   
mismatch
镜像, ring内整体轮换两次
[C][N][NH1][NH1][O][NH1][Ring1][Branch1]

22  /opt/huangyan/mol-ai/images/ring_selfies/length_8/191.png   {
"selfies": "[C][O][N][N][C][Ring1][Branch1][C]"}         C1ONNC1C   {
"selfies": "[C][C][O][N][N][Ring1][Branch1][C]"}  C1CONN1C    0.185185   
mismatch
ring内整体轮换

15  /opt/huangyan/mol-ai/images/ring_selfies/length_8/140.png     {
"selfies": "[O][C][N][N][C][Ring1][Ring1][C]"}         OCN1NC1C     {
"selfies": "[C][N][C][N][Ring1][Ring1][C][O]"}  CN1CN1CO    0.260870   
mismatch
镜像, 4-5轮换(ring内部分轮换, 靠近ring op的两原子)
  [C][N][C][N][Ring1][Ring1][C][O]
  [N][Branch1][C][C][C][N][Ring1][Ring2][C][O]
  [C][N][Branch1][C][C][N][Ring1][Ring2][C][O]
  [N][Branch1][Ring1][C][O][C][N][Ring1][Branch1][C]
  [C][Branch1][C][O][N][C][N][Ring1][Ring1][C]
  [O][C][N][C][N][Ring1][Ring1][C]

24  /opt/huangyan/mol-ai/images/ring_selfies/length_8/107.png  {
"selfies": "[O][C][O][O][O][C][Ring1][=Branch1]"}         O1COOOC1  {
"selfies": "[O][C][O][O][C][O][Ring1][=Branch1]"}  O1COOCO1    0.333333   
mismatch
5-6轮换(ring内部分轮换, 靠近ring op的两原子)

``` 

  

后续可改进的方向, 增加靠近ring op的原子的相邻轮换 (需要继续观察是否是一种明显的规律)

  

# 训练10

进一步提高branch识别的效率, 进行如下训练: 

  - 阶段1: 

    - 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4; 

    - 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 2倍的ring平移数据, 等量的在ring环内进行整段轮换的数据

    - 长度5-8的branch原子, 带2倍的基于branch分子的rotate训练数据, 掺入等量的branch增强数据(添加[branch]操作符), 掺入2倍的改变普通分子起点形成的branch分子 (注意长度是5-10),

  

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) |
| --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 94.74% |
| 基础原子, 长度8 | 100.00% | 100.00% |
| ring分子, 长度7 | 95.00% | 95.00% |
| ring分子, 长度8 | 95.00% | 100.00% |
| branch分子, 长度7 | 76.67% | 86.67% |
| branch分子, 长度8 | 90.00% | 83.33% |
  
  

分析标红处的错误: 

```
 2    /opt/huangyan/mol-ai/images/branch_selfies/length_7/56.png  {
"selfies": "[C][Branch1][Branch1][O][N][O][C]"}            CONOC  {
"selfies": "[C][Branch1][Ring2][O][N][O][C]"}   C(ONO)C    0.166667   
mismatch
分支长度错误

18   /opt/huangyan/mol-ai/images/branch_selfies/length_7/20.png    {
"selfies": "[C][Branch1][Ring1][N][C][O][O]"}          C(NC)OO  {
"selfies": "[N][Branch1][Ring1][C][O][C][O]"}   N(CO)CO    0.166667   
mismatch
复杂变化
[N][Branch1][Ring1][C][O][C][O]
[C][Branch1][C][O][N][C][O]
[O][C][N][C][O]
[C][Branch1][C][O][N][C][O]
[O][C][N][C][O]

24  /opt/huangyan/mol-ai/images/branch_selfies/length_7/104.png    {
"selfies": "[C][Branch1][Ring1][C][N][C][N]"}          C(CN)CN  {
"selfies": "[N][Branch1][Ring1][C][N][C][C]"}   N(CN)CC    0.187500   
mismatch
1,7对调

23   /opt/huangyan/mol-ai/images/branch_selfies/length_7/66.png    {
"selfies": "[O][Branch1][Ring2][N][C][C][C]"}          O(NCC)C      {
"selfies": "[N][Branch1][C][C][O][C][C]"}   N(C)OCC    0.263158   
mismatch
起点3, 分支长度错误, 分支内轮换*2
[O][Branch1][Ring1][C][C][N][C]

8   /opt/huangyan/mol-ai/images/branch_selfies/length_7/126.png        {
"selfies": "[C][Branch1][C][O][C][O][N]"}          C(O)CON  {
"selfies": "[O][Branch1][Ring1][C][N][C][O]"}   O(CN)CO    0.263158   
mismatch
起点2, 4-7轮换 (分支内外轮换)
[C][Branch1][C][N][O][C][O]

17  /opt/huangyan/mol-ai/images/branch_selfies/length_7/131.png    {
"selfies": "[C][Branch1][Ring1][N][C][O][C]"}          C(NC)OC  {
"selfies": "[C][Branch1][Ring2][C][N][C][O]"}   C(CNC)O    0.263158   
mismatch
分支长度错误, 4-7轮换 (分支内外轮换)

27  /opt/huangyan/mol-ai/images/branch_selfies/length_7/124.png    {
"selfies": "[C][C][Branch1][Ring1][C][O][C]"}          CC(CO)C      {
"selfies": "[C][C][Branch1][C][O][C][C]"}   CC(O)CC    0.357143   
mismatch
分支长度错误, 5-7轮换 (分支内外轮换)

``` 

  

```
10    /opt/huangyan/mol-ai/images/branch_selfies/length_7/1.png  {
"selfies": "[O][Branch1][Ring1][N][O][C][N]"}          O(NO)CN  {
"selfies": "[O][Branch1][Ring2][N][O][C][N]"}   O(NOC)N    0.136364   
mismatch
分支长度错误

20   /opt/huangyan/mol-ai/images/branch_selfies/length_7/67.png  {
"selfies": "[C][Branch1][Ring2][N][C][N][O]"}          C(NCN)O  {
"selfies": "[C][Branch1][Ring1][N][C][N][O]"}   C(NC)NO    0.150000   
mismatch
分支长度错误

24  /opt/huangyan/mol-ai/images/branch_selfies/length_7/104.png  {
"selfies": "[C][Branch1][Ring1][C][N][C][N]"}          C(CN)CN  {
"selfies": "[N][Branch1][Ring1][C][N][C][C]"}   N(CN)CC    0.187500   
mismatch
1,7对调

17  /opt/huangyan/mol-ai/images/branch_selfies/length_7/131.png  {
"selfies": "[C][Branch1][Ring1][N][C][O][C]"}          C(NC)OC  {
"selfies": "[C][Branch1][Ring2][C][N][C][O]"}   C(CNC)O    0.263158   
mismatch
分支长度错误, 4-7轮换

``` 

  

主要错误: 

  - 分支长度错误

  - ring op后 一直到 结尾的轮换

# 训练11

增强对分支长度的理解, 改造"掺入等量的branch增强数据(添加[branch]操作符)-v2", 将一个base selfies生成多个branch长度 (3个), 进行如下训练: 

  - 阶段1: 

    - 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4; 

    - 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 2倍的ring平移数据, 等量的在ring环内进行整段轮换的数据

    - 长度5-8的branch原子, 带2倍的基于branch分子的rotate训练数据, 掺入等量的branch增强数据(添加[branch]操作符)-v2, 掺入2倍的改变普通分子起点形成的branch分子 (注意长度是5-10)

  

训练日志: [run_tasks.2329.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/run_tasks.2329.log)

训练结果: 

  

掺入等量的branch增强数据(添加[branch]操作符)-v2

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=6) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 94.74% | 100.00% | 100.00% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| ring分子, 长度7 | 95.00% | 95.00% | 95.00% |
| ring分子, 长度8 | 87.50% | 90.00% | 95.00% |
| branch分子, 长度7 | 76.67% | 93.33% | 96.67% |
| branch分子, 长度8 | 86.67% | 93.33% | 100.00% |
  
掺入2倍的branch增强数据(添加[branch]操作符)-v2

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=6) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 100.00% |
| 基础原子, 长度8| 100.00%| 100.00%| 100.00%
| ring分子, 长度7| 97.50%| 100.00%| 97.50%
| ring分子, 长度8| 80.00%| 85.00%| 87.50%
| branch分子, 长度7| 90.00%| 96.67%| 93.33%
| branch分子, 长度8| 86.67%| 86.67%| 90.00%
  
branch的效果得到改善:

  - 但当branch的增强数据为2倍时, 不如1倍时的效果. 
  - 在1倍数据的步骤1中: ring的效果也有下降. branch的错误也需要分析

1倍数据的 (branch分子, 长度7) 错误分析: 

```
 9    /opt/huangyan/mol-ai/images/branch_selfies/length_7/40.png  {
"selfies": "[C][Branch1][Ring2][C][C][N][O]"}          C(CCN)O      {
"selfies": "[N][Branch1][C][O][C][C][C]"}   N(O)CCC    0.095238   
mismatch
起点4, branch长度错误
[C][Branch1][C][C][C][N][O]

7    /opt/huangyan/mol-ai/images/branch_selfies/length_7/65.png  {
"selfies": "[O][Branch1][Ring1][N][C][N][O]"}          O(NC)NO      {
"selfies": "[C][Branch1][C][N][N][O][O]"}   C(N)NOO    0.136364   
mismatch
起点4, branch长度错误, 4-7轮换(分支内外轮换)
[O][Branch1][C][O][N][C][N]

23   /opt/huangyan/mol-ai/images/branch_selfies/length_7/20.png  {
"selfies": "[C][Branch1][Ring1][N][C][O][O]"}          C(NC)OO  {
"selfies": "[N][Branch1][Ring1][C][O][C][O]"}   N(CO)CO    0.166667   
mismatch
起点4, branch长度错误, 4-7轮换(分支内外轮换)
[C][Branch1][C][O][N][C][O]

28    /opt/huangyan/mol-ai/images/branch_selfies/length_7/6.png  {
"selfies": "[C][Branch1][Ring1][O][N][C][N]"}          C(ON)CN  {
"selfies": "[O][Branch1][Ring2][N][C][N][C]"}   O(NCN)C    0.200000   
mismatch
起点4, branch长度错误, 复杂变换
[C][Branch1][C][N][N][O][C]

3    /opt/huangyan/mol-ai/images/branch_selfies/length_7/77.png  {
"selfies": "[C][Branch1][Ring1][C][N][O][N]"}          C(CN)ON      {
"selfies": "[N][Branch1][C][O][C][C][N]"}   N(O)CCN    0.210526   
mismatch
起点3, 6-7轮换
[C][Branch1][Ring1][C][N][N][O]

0    /opt/huangyan/mol-ai/images/branch_selfies/length_7/83.png  {
"selfies": "[N][Branch1][Ring1][O][N][N][N]"}          N(ON)NN  {
"selfies": "[O][Branch1][Ring1][N][N][N][N]"}   O(NN)NN    0.266667   
mismatch
起点4, branch长度错误, 4-5轮换
[N][Branch1][C][N][O][N][N]

20  /opt/huangyan/mol-ai/images/branch_selfies/length_7/124.png  {
"selfies": "[C][C][Branch1][Ring1][C][O][C]"}          CC(CO)C      {
"selfies": "[C][C][Branch1][C][O][C][C]"}   CC(O)CC    0.357143   
mismatch
branch长度错误, 5-6轮换

``` 

  

1倍数据的 (ring分子, 长度8) 错误分析: 

```
 22   /opt/huangyan/mol-ai/images/ring_selfies/length_8/93.png   {
"selfies": "[O][N][C][N][N][N][Ring1][Branch1]"}         ON1CNNN1     {
"selfies": "[O][N][C][N][N][N][Ring1][Ring2]"}  ONC1NNN1    0.120000   
mismatch
ring长度错误, ring在末尾

34   /opt/huangyan/mol-ai/images/ring_selfies/length_8/20.png   {
"selfies": "[O][N][N][C][N][Ring1][Branch1][O]"}         O1NNCN1O   {
"selfies": "[O][N][N][C][N][O][Ring1][Branch1]"}  ON1NCNO1    0.280000   
mismatch
镜像, 3-5轮换(环中后半部分轮换)
[O][N][C][N][N][Ring1][Branch1][O]

8   /opt/huangyan/mol-ai/images/ring_selfies/length_8/193.png   {
"selfies": "[N][N][C][O][C][Ring1][Branch1][N]"}         N1NCOC1N   {
"selfies": "[C][N][N][O][C][Ring1][Branch1][N]"}  C1NNOC1N    0.280000   
mismatch
1-3轮换(环中前半部分轮换)

13  /opt/huangyan/mol-ai/images/ring_selfies/length_8/124.png     {
"selfies": "[N][O][C][C][C][N][Ring1][Ring2]"}         NOC1CCN1     {
"selfies": "[N][O][C][C][N][C][Ring1][Ring2]"}  NOC1CNC1    0.318182   
mismatch
5-6轮换(环中后半部分轮换)

7    /opt/huangyan/mol-ai/images/ring_selfies/length_8/90.png     {
"selfies": "[O][C][N][N][Ring1][Ring2][N][C]"}         O1CNN1NC     {
"selfies": "[C][N][N][C][N][O][Ring1][Ring2]"}  CNN1CNO1    0.333333   
mismatch
镜像, 2-4轮换(环中后半部分轮换)
[O][N][C][N][Ring1][Ring2][N][C]
``` 

  

主要错误: 

  - branch: branch长度错误
  - ring: 环中 前半部分/后半部分 轮换

  

将rotate_ring_atoms和add_branch_op_from_random_selfies_only_basic_atom 增加数据量, 争取在一个步骤内能达标

# 训练12

将rotate_ring_atoms和add_branch_op_from_random_selfies_only_basic_atom 增加数据量, 争取在一个步骤内能达标, 进行如下训练: 

  - 阶段1: 

    - 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4; 

    - 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 2倍的ring平移数据, 掺入2倍的在ring环内进行整段轮换的数据

    - 长度5-8的branch原子, 带2倍的基于branch分子的rotate训练数据, 掺入3倍的branch增强数据(添加[branch]操作符)-v2, 掺入2倍的改变普通分子起点形成的branch分子 (注意长度是5-10)

commit: 2678faf742b000f35953460d4959eb67acf880b0

训练日志: [run_tasks.0959.log](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/run_tasks.0959.log)

训练效果:

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=6) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 94.74% | 89.47% | 100.00% |
| 基础原子, 长度8 | 100.00% | 94.74% | 100.00% |
| ring分子, 长度7 | 97.50% | 95.00% | 100.00% |
| ring分子, 长度8 | 90.00% | 100.00% | 97.50% |
| branch分子, 长度7 | 86.67% | 86.67% | 96.67% |
| branch分子, 长度8 | 90.00% | 93.33% | 93.33% |
  
步骤1中, 先提高branch分子的识别率. 分析错误: 

branch分子, 长度7的错误分布: 

```
 13  /opt/huangyan/mol-ai/images/branch_selfies/length_7/103.png  {
"selfies": "[C][Branch1][Ring1][N][N][C][O]"}          C(NN)CO  {
"selfies": "[N][Branch1][Ring1][N][C][C][O]"}   N(NC)CO    0.200000   
mismatch
[O][C][C][N][N]和[O][C][N][N][C]的区别

15  /opt/huangyan/mol-ai/images/branch_selfies/length_7/142.png  {
"selfies": "[N][Branch1][Ring1][N][N][O][N]"}          N(NN)ON      {
"selfies": "[N][Branch1][C][O][N][N][N]"}   N(O)NNN    0.222222   
mismatch
[N][O][N][N][N]和[N][N][N][N][O]的区别

0    /opt/huangyan/mol-ai/images/branch_selfies/length_7/83.png  {
"selfies": "[O][Branch1][Ring2][N][N][N][N]"}          O(NNN)N  {
"selfies": "[O][Branch1][Ring1][N][N][N][N]"}   O(NN)NN    0.266667   
mismatch
分支长度错误

4   /opt/huangyan/mol-ai/images/branch_selfies/length_7/101.png  {
"selfies": "[N][Branch1][Ring1][O][N][N][N]"}          N(ON)NN      {
"selfies": "[N][Branch1][C][N][O][N][N]"}   N(N)ONN    0.266667   
mismatch
[N][N][N][O][N]和[N][N][O][N][N]的区别

``` 

branch分子, 长度8的错误分布: 

```
17    /opt/huangyan/mol-ai/images/branch_selfies/length_8/4.png    {
"selfies": "[C][Branch1][Ring1][O][C][N][N][O]"}         C(OC)NNO  {
"selfies": "[C][Branch1][Branch1][O][C][N][N][O]"}  C(OCNN)O    0.200000   
mismatch
[O][N][N][C][O][C] 和 [O][C][O][C][N][N] 的区别

11   /opt/huangyan/mol-ai/images/branch_selfies/length_8/88.png        {
"selfies": "[N][Branch1][C][C][C][O][C][N]"}         N(C)COCN    {
"selfies": "[C][Branch1][Ring2][N][C][N][O][C]"}  C(NCN)OC    0.304348   
mismatch
[N][C][O][C][N][C]和[C][O][C][N][C][N]的区别

15   /opt/huangyan/mol-ai/images/branch_selfies/length_8/37.png        {
"selfies": "[N][Branch1][C][O][C][N][C][O]"}         N(O)CNCO    {
"selfies": "[C][Branch1][Ring1][O][N][N][C][O]"}  C(ON)NCO    0.333333   
mismatch
[O][C][N][C][N][O] 和 [O][C][N][C][O][N]的区别
``` 

对直链分子的变化 (相邻分子交换 + 整体轮换) 需要增强理解, 在rotate_selfies_substring_from_random_selfies_only_basic_atom的分子变化基础上, 变成branch

  

# 训练12-额外测试batch_size

在训练12的基础上, batch_size设为4

训练时间: 2120 seconds (训练12的时间是 2467 seconds)

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) | 步骤3 (epoch=6) (LR=3e-05) |
| --- | --- | --- | --- |
| 基础原子, 长度7 | 100.00% | 100.00% | 94.74% |
| 基础原子, 长度8 | 100.00% | 100.00% | 100.00% |
| ring分子, 长度7 | 97.50% | 95.00% | 97.50% |
| ring分子, 长度8 | 87.50% | 100.00% | 100.00% |
| branch分子, 长度7 | 93.33% | 90.00% | 90.00% |
| branch分子, 长度8 | 80.00% | 93.33% | 90.00% |
  
  

并没有看到明显的收敛或者性能提升, 其中步骤1的(branch分子, 长度8)性能下降

  

# 训练13

增加branch_selfies_by_diff_start_point_from_rotate_selfies_substring_from_random_selfies_only_basic_atom (直链分子进行局部rotate并形成branch表达式), 争取在一个步骤内能达标, 进行如下训练: 

  - 阶段1: 

    - 长度3-8的基础原子, 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v4; 

    - 长度5-8的ring原子, 带等量的基于ring分子的rotate训练数据, 2倍的ring平移数据, 掺入2倍的在ring环内进行整段轮换的数据

    - 长度5-8的branch原子, 带2倍的基于branch分子的rotate训练数据, 掺入3倍的branch增强数据(添加[branch]操作符)-v2, 掺入2倍的改变普通分子起点形成的branch分子 (注意长度是5-10), 掺入等量的直链分子进行局部rotate并形成branch表达式

commit: 349d109dd34288163b6010d78fcc62fb5a419d6b

训练效果: 

|  | 步骤1 (epoch=3) (LR=1e-4) | 步骤2 (epoch=6) (LR=5e-05) |
| --- | --- | --- |
| 基础原子, 长度7 | 94.74% | 94.74% |
| 基础原子, 长度8 | 100.00% | 100.00% |
| ring分子, 长度7 | 97.50% | 100.00% |
| ring分子, 长度8 | 95.00% | 95.00% |
| branch分子, 长度7 | 90.00% | 96.67% |
| branch分子, 长度8 | 76.67% | 86.67% |
  
  

branch准确率没有明确提升, 对错误进行分析: 

对(branch分子, 长度8)的错误分析: 

```
步骤1: 
 
15  /opt/huangyan/mol-ai/images/branch_selfies/length_8/134.png    {
"selfies": "[C][C][C][Branch1][Ring1][N][N][C]"}         CCC(NN)C    {
"selfies": "[C][N][N][Branch1][Ring1][C][C][C]"}  CNN(CC)C    0.173913   
mismatch
复杂变化, 表面上看, 是整个分子镜像(但branch op没有变)

18  /opt/huangyan/mol-ai/images/branch_selfies/length_8/137.png           {
"selfies": "[N][Branch1][C][O][N][C][C]"}          N(O)NCC    {
"selfies": "[N][Branch1][Ring2][C][N][C][C][O]"}  N(CNC)CO    0.190476   
mismatch
[C][C][N][N][O]和[C][N][C][N][C][O]的区别, 原子长度错误

2   /opt/huangyan/mol-ai/images/branch_selfies/length_8/102.png        {
"selfies": "[N][C][N][Branch1][C][O][C][N]"}         NCN(O)CN    {
"selfies": "[O][C][Branch1][Ring2][N][C][N][N]"}  OC(NCN)N    0.200000   
mismatch
镜像, 整个分支左移一位
[N][C][Branch1][C][O][N][C][N]

0   /opt/huangyan/mol-ai/images/branch_selfies/length_8/109.png    {
"selfies": "[N][Branch1][Ring2][C][C][C][N][O]"}         N(CCC)NO    {
"selfies": "[N][Branch1][Ring1][C][C][C][N][O]"}  N(CC)CNO    0.272727   
mismatch
分支长度错误

23  /opt/huangyan/mol-ai/images/branch_selfies/length_8/126.png    {
"selfies": "[C][O][N][Branch1][Ring1][N][C][O]"}         CON(NC)O    {
"selfies": "[O][N][N][Branch1][Ring1][O][C][C]"}  ONN(OC)C    0.333333   
mismatch
起点5, 分支长度错误, 6-7轮换 (分支内外轮换)
[C][O][N][Branch1][C][C][N][O]

19  /opt/huangyan/mol-ai/images/branch_selfies/length_8/101.png        {
"selfies": "[N][Branch1][C][C][C][O][O][N]"}         N(C)COON    {
"selfies": "[O][Branch1][Ring1][O][N][C][C][N]"}  O(ON)CCN    0.333333   
mismatch
[N][O][O][C][N][C]和[N][O][O][C][C][N]的差异

11   /opt/huangyan/mol-ai/images/branch_selfies/length_8/32.png    {
"selfies": "[N][C][Branch1][Ring1][O][N][O][O]"}         NC(ON)OO        {
"selfies": "[O][N][C][Branch1][C][N][O][O]"}  ONC(N)OO    0.350000   
mismatch
起点4, 5-6轮换(分支内轮换)
[N][C][Branch1][Ring1][N][O][O][O]
 
 
------
 
步骤2: 
2   /opt/huangyan/mol-ai/images/branch_selfies/length_8/102.png        {
"selfies": "[N][C][N][Branch1][C][C][N][O]"}         NCN(C)NO    {
"selfies": "[O][C][Branch1][Ring2][N][C][N][N]"}  OC(NCN)N    0.208333   
mismatch
长度5, 整个分支向右平移一位
[N][C][N][C][Branch1][C][N][O]

23  /opt/huangyan/mol-ai/images/branch_selfies/length_8/126.png    {
"selfies": "[C][O][N][Branch1][Ring1][N][C][O]"}         CON(NC)O    {
"selfies": "[O][N][N][Branch1][Ring1][O][C][C]"}  ONN(OC)C    0.333333   
mismatch
长度5, 分支长度错误, 分支内原子轮换
[C][O][N][Branch1][C][C][N][O]

11   /opt/huangyan/mol-ai/images/branch_selfies/length_8/32.png    {
"selfies": "[N][C][Branch1][Ring1][O][N][O][O]"}         NC(ON)OO        {
"selfies": "[O][N][C][Branch1][C][N][O][O]"}  ONC(N)OO    0.350000   
mismatch
长度4, 分支内原子轮换
[N][C][Branch1][Ring1][N][O][O][O]

14  /opt/huangyan/mol-ai/images/branch_selfies/length_8/116.png        {
"selfies": "[C][N][N][Branch1][C][O][N][C]"}         CNN(O)NC        {
"selfies": "[O][N][N][Branch1][C][C][N][C]"}  ONN(C)NC    0.352941   
mismatch
镜像, 6-8镜像 (分支内外镜像)
[C][N][N][Branch1][C][C][N][O]

``` 

  

对(branch分子, 长度8)的错误分析: 

```
 27   /opt/huangyan/mol-ai/images/branch_selfies/length_7/44.png  {
"selfies": "[N][Branch1][Ring1][O][N][O][O]"}          N(ON)OO  {
"selfies": "[O][Branch1][Ring1][N][O][N][O]"}   O(NO)NO    0.176471   
mismatch
[O][O][N][O][N] 和 [O][N][O][N][O]的区别

14   /opt/huangyan/mol-ai/images/branch_selfies/length_7/69.png      {
"selfies": "[N][Branch1][C][O][O][N][N]"}          N(O)ONN  {
"selfies": "[N][Branch1][Ring1][O][O][N][N]"}   N(OO)NN    0.263158   
mismatch
分支长度错误

15   /opt/huangyan/mol-ai/images/branch_selfies/length_7/56.png  {
"selfies": "[O][Branch1][Ring2][N][C][C][O]"}          O(NCC)O  {
"selfies": "[C][Branch1][Ring2][O][N][O][C]"}   C(ONO)C    0.300000   
mismatch
[C][C][N][O][O] 和 [C][C][O][N][O]的区别

``` 

  

对直链分子的区分仍需提高

  

# 训练14

改造 branch_selfies_by_diff_start_point_from_rotate_selfies_substring_from_random_selfies_only_basic_atom (直链分子进行局部rotate并形成branch表达式), 将rotate前后的表达式都加入. 效果仍然不好. 无法在一个步骤内提升准确率.

做了一些尝试: 

  1. 调整不同策略, 发现步骤1的结果都不会太好, 但步骤3的结果都还可以.
  2. 怀疑是LR的原因, 因为每个步骤开始会重置LR. 尝试降低每个步骤的epoch数量, 让LR重置频率变高

以上尝试都没有很好的效果.

跑一组去掉branch_selfies_by_diff_start_point_from_rotate_selfies_substring_from_random_selfies_only_basic_atom的基线, 然后研究基线上的Lora层的权重分布, 进行以下尝试:

  

基线:

commit: b499dc1cec38c06f88a80aff50b7078dadf3942b

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

  

三个步骤的checkpoint:

/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v817-20250212-002052/checkpoint-801  
/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v818-20250212-010602/checkpoint-801  
/opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v819-20250212-015105/checkpoint-801

  

  

期望是能将lora层可视化, 通过对比几根基线, 看到结果的某一部分跟lora的某一些层正相关, 这样可以通过调配部分数据的重复或扩增, 明确改善效果. 而不是现在这样盲猜.

commit: f51aabfc00f3ad08dfb30066a04a2e1b46a9bfb7

增加lora层的可视化, 将"原始checkpoint - 步骤1 - 步骤2 - 步骤3" 逐步进行对比: 

![image2025-2-12 11:8:40.png](/assets/01KJBZRBSBD6C78ZNC9ERG9QB3/image2025-2-12%2011%3A8%3A40.png) 

  

可以看到: 

  1. 步骤3的主要变化在layer 6-10, 认为这部分与 (基础原子, 长度7) 高度相关

  

接下来放在单独的文档里, 研究训练数据和lora层的关系
