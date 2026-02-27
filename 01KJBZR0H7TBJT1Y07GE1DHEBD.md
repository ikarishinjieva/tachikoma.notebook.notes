---
title: 20250131 - 使用ms-swift对Qwen2-VL进行微调 [21] - 训练一个识别基础原子的基模型
confluence_page_id: 3344058
created_at: 2025-01-31T08:46:08+00:00
updated_at: 2025-02-07T05:58:26+00:00
---

# 回顾

[20250127 - 使用ms-swift对Qwen2-VL进行微调 [20] - 调优长度8的效果[1]]

在长度8的预测中, 出现了许多键错误, 增加 针对性的数据增强后, 效果并不好. 

但训练时间已经过长, 先想办法降低训练时间. 

于是想排除 ring和branch,训练一个识别基础院子的基模型

# 训练方式1

随机生成的分子:

  - 长度3+长度4
  - 长度3+长度4+长度5
  - 长度3+长度4+长度5+长度6
  - 长度3+长度4+长度5+长度6+长度7

每一步进行1个epoch, 一共四步, 步骤4的训练效果: 

```
评估长度5:
Correct: 88.89%, error_type
            88.888889
mismatch    11.111111

评估长度6:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度7:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估长度8:
Correct: 26.32%, error_type
mismatch    73.684211
            26.315789
``` 

推理能力很低

# 训练方式2

commit: ad3b14ed10f68a5e6389b349a03a698808b3df91

随机生成的分子:

  - 长度3+长度4
  - 长度3+长度4+长度5
  - 长度3+长度4+长度5+长度6
  - 长度3+长度4+长度5+长度6+长度7

每一步进行2个epoch, 除最长的长度数据为100个, 其他长度数据为50个, 训练效果: 

```
# 步骤1

评估长度5:
Correct: 88.89%, error_type
            88.888889
mismatch    11.111111

评估长度6:
Correct: 88.89%, error_type
            88.888889
mismatch    11.111111

评估长度7:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789

评估长度8:
Correct: 31.58%, error_type
mismatch    68.421053
            31.578947

# 步骤2

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估长度8:
Correct: 47.37%, error_type
mismatch    52.631579
            47.368421

# 步骤3

评估长度5:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 78.95%, error_type
            84.210526
mismatch    15.789474

评估长度8:
Correct: 57.89%, error_type
            57.894737
mismatch    42.105263

# 步骤4

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度8:
Correct: 31.58%, error_type
mismatch    68.421053
            31.578947

# 步骤5

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估长度8:
Correct: 26.32%, error_type
mismatch    73.684211
            26.315789

# 步骤6

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估长度8:
Correct: 47.37%, error_type
mismatch    52.631579
            47.368421

# 步骤7

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 84.21%, error_type
            89.473684
mismatch    10.526316

评估长度8:
Correct: 42.11%, error_type
mismatch    57.894737
            42.105263

# 步骤8

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 78.95%, error_type
            84.210526
mismatch    15.789474

评估长度8:
Correct: 31.58%, error_type
mismatch    68.421053
            31.578947 
``` 

SOTA出现在步骤3 (epoch=6): 57.89%

# 训练方式3

随机生成的分子:

  - 长度3(50个)+长度4 (100个)
  - 长度4(50个)+长度5 (100个)
  - 长度5(50个)+长度6 (100个)
  - 长度6(50个)+长度7 (100个)

每一步进行2个epoch, 训练效果: 

```
# 步骤1

评估长度5:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度6:
Correct: 83.33%, error_type
            83.333333
mismatch    16.666667

评估长度7:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估长度8:
Correct: 36.84%, error_type
mismatch    63.157895
            36.842105

# 步骤2

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度8:
Correct: 31.58%, error_type
mismatch    68.421053
            31.578947

# 步骤3

评估长度5:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度8:
Correct: 31.58%, error_type
mismatch    68.421053
            31.578947

# 步骤4

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 88.89%, error_type
            88.888889
mismatch    11.111111

评估长度7:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估长度8:
Correct: 36.84%, error_type
mismatch    63.157895
            36.842105

``` 

推理能力下降

# 训练方式4

随机生成的分子:

  - 长度3(50个)+长度4 (100个)
  - 长度3(50个)+长度4(50个)+长度5 (100个)
  - 长度4(50个)+长度5(50个)+长度6 (100个)
  - 长度5(50个)+长度6(50个)+长度7 (100个)

每一步进行2个epoch, 训练效果: 

```
 # 步骤1

评估长度5:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度6:
Correct: 88.89%, error_type
            88.888889
mismatch    11.111111

评估长度7:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估长度8:
Correct: 31.58%, error_type
mismatch    68.421053
            31.578947

# 步骤2

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估长度8:
Correct: 31.58%, error_type
mismatch    68.421053
            31.578947

# 步骤3

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估长度8:
Correct: 36.84%, error_type
mismatch    63.157895
            36.842105
``` 

推理能力仍很低

# 训练方式5

随机生成的分子:

  - 长度3(50个)+长度4 (100个)
  - 长度3(25个)+长度4(50个)+长度5 (100个)
  - 长度3(12个)+长度4(25个)+长度5(50个)+长度6 (100个)
  - 长度3(6个)+长度4(12个)+长度5(25个)+长度6(50个)+长度7 (100个)

使用递减数据量

每一步进行2个epoch, 训练效果: 

```
# 步骤1

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 88.89%, error_type
            88.888889
mismatch    11.111111

评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度8:
Correct: 26.32%, error_type
mismatch    73.684211
            26.315789

# 步骤2

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度8:
Correct: 26.32%, error_type
mismatch    73.684211
            26.315789

# 步骤3

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度8:
Correct: 36.84%, error_type
mismatch    63.157895
            36.842105
``` 

推理能力仍很低

# 训练方式6

随机生成的分子:

  - 长度3(50个)+长度4 (100个)
  - 长度3(25个)+长度4(50个)+长度5 (100个)
  - 长度3(12个)+长度4(25个)+长度5(50个)+长度6 (100个)
  - 长度3(6个)+长度4(12个)+长度5(25个)+长度6(50个)+长度7 (100个)

使用递减数据量

每一步进行3个epoch, 训练效果: 

```
# 步骤1

评估长度5:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度6:
Correct: 88.89%, error_type
            88.888889
mismatch    11.111111

评估长度7:
Correct: 63.16%, error_type
            63.157895
mismatch    36.842105

评估长度8:
Correct: 52.63%, error_type
            52.631579
mismatch    47.368421

# 步骤2

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度8:
Correct: 57.89%, error_type
            57.894737
mismatch    42.105263

# 步骤3

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度8:
Correct: 63.16%, error_type
            63.157895
mismatch    36.842105 
``` 

推理能力提升到 63.16%

对错误进行分析: 

  - 长度7的错误都出现在 镜像
  - 长度8的错误出现在 原子交换 或 缺原子

# 训练方式7

随机生成的分子:

  - 长度3(50个)+长度4 (100个)
  - 长度3(25个)+长度4(50个)+长度5 (100个)
  - 长度3(12个)+长度4(25个)+长度5(50个)+长度6 (100个)
  - 长度3(6个)+长度4(12个)+长度5(25个)+长度6(50个)+长度7 (100个)

使用递减数据量

每一步进行4个epoch, 训练效果: 

```
# 步骤1

评估长度5:
Correct: 88.89%, error_type
            88.888889
mismatch    11.111111

评估长度6:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度7:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估长度8:
Correct: 21.05%, error_type
mismatch    78.947368
            21.052632

# 步骤2

评估长度5:
Correct: 94.44%, error_type
            94.444444
mismatch     5.555556

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估长度8:
Correct: 52.63%, error_type
            52.631579
mismatch    47.368421

# 步骤3

评估长度5:
Correct: 83.33%, error_type
            83.333333
mismatch    16.666667

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789

评估长度8:
Correct: 52.63%, error_type
            52.631579
mismatch    47.368421
 
``` 

训练效果下降

# 训练方式8

随机生成的分子:

  - 长度3(50个)+长度4 (100个), 带等量的镜像增强数据
  - 长度3(25个)+长度4(50个)+长度5 (100个), 带等量的镜像增强数据
  - 长度3(12个)+长度4(25个)+长度5(50个)+长度6 (100个), 带等量的镜像增强数据
  - 长度3(6个)+长度4(12个)+长度5(25个)+长度6(50个)+长度7 (100个), 带等量的镜像增强数据

commit: 13879425052af93d333a2356014b69914cdc5dea

训练效果: 

```
 # 步骤1

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估长度8:
Correct: 57.89%, error_type
            57.894737
mismatch    42.105263

# 步骤2

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估长度8:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

# 步骤3

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 100.00%, error_type
    100.0

评估长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

# 步骤4

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 100.00%, error_type
    100.0

评估长度8:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

# 步骤5

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估长度8:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789

# 步骤6

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估长度8:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789

# 步骤7

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

# 步骤8

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估长度8:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

``` 

增强数据能提高长度8的效果

评估: 步骤2的长度8的效果: 错误都是将某个原子抽取后, 插入回另外一个原子位置: [8.analyzed.0211.log](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/8.analyzed.0211.log)

SOTA出现在步骤3和步骤7, 分析错误类型:

  - 步骤3, 长度8: 问题都是原子交换 (或轮换)
  - 步骤7, 长度7+长度8: 四个问题, 三个是原子交换, 一个是原子插入 (或者都归纳为 局部的轮换)

增加长度9的评估, 分析错误分类: [length_9.analyzed.1220.log](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/length_9.analyzed.1220.log) :

```
两次轮换: 3-8轮换, 4-5轮换
两次轮换: 4-5轮换, 6-7轮换

镜像([C][N][C][C][O][N][C][N][C]), 4-5轮换
镜像([N][C][N][N][C][N][O][C][N]), 6-9轮换
镜像([N][N][N][O][N][O][O][C][N]), 5-9轮换
镜像([O][C][N][C][O][O][N][O][C]), 4-7轮换
镜像([N][N][O][O][C][C][N][O][O]), 4-6轮换
镜像([O][O][C][O][N][O][C][C][C]), 1-8轮换
 
5-7轮换
5-7轮换
5-8轮换
 
少原子[N], 5-6轮换
少一个[C]原子
``` 

# 训练方式9

增加增强数据: 对于分子中某一子段进行轮换

随机生成的分子:

  - 长度3(50个)+长度4 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据
  - 长度3(25个)+长度4(50个)+长度5 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据
  - 长度3(12个)+长度4(25个)+长度5(50个)+长度6 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据
  - 长度3(6个)+长度4(12个)+长度5(25个)+长度6(50个)+长度7 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据

commit: ec0d8b099776fa852f4dab95c7fb98acf844c952

训练效果: 

```
# 步骤1

评估长度5:
Correct: 100.00%, error_type
    100.0

评估长度6:
Correct: 100.00%, error_type
    100.0

评估长度7:
Correct: 100.00%, error_type
    100.0

评估长度8:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474 
 
# 步骤2/3 跟步骤1的正确率类似
``` 

长度8的错误分析: 2个原子交换, 1个缺原子

增加长度9的评估: 正确率从 7/20 提升到 12/20: [length_9.analyzed.2257.log](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/length_9.analyzed.2257.log), 错误类型:

```
镜像([C][C][O][O][N][C][N][C][O]) + 原子交换
镜像([N][O][C][C][N][C][N][O][N]) + 原子交换
镜像([N][C][O][N][O][O][C][C][N]) + 原子交换
原子交换
原子交换

镜像([N][C][O][C][O][N][N][N][O]) + 原子交换 + 少原子
镜像([O][C][O][N][C][O][O][C][C]) + 少原子
少原子
``` 

原子交换的位置都集中于4和8左右

重新制作 "子段轮换的增强数据": 

  - 都做成原子相邻交换
  - 位置以4和8为主
  - 为镜像selfies也生成增强数据

# 训练方式10

重新制作 "子段轮换的增强数据-v2": 

  - 都做成原子相邻交换
  - 位置以4和8为主
  - 为镜像selfies也生成增强数据

随机生成的分子:

  - 长度3(50个)+长度4 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v2
  - 长度3(25个)+长度4(50个)+长度5 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v2
  - 长度3(12个)+长度4(25个)+长度5(50个)+长度6 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v2
  - 长度3(6个)+长度4(12个)+长度5(25个)+长度6(50个)+长度7 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v2

commit: 9483d00d5cee98bf0c3b8f8a3880ad5bb2d8430e

训练效果: 

```
# 步骤1

评估长度7:
Correct: 100.00%, error_type
    100.0

评估长度8:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度9:
Correct: 45.00%, error_type
mismatch    55.0
            45.0
 
 
# 步骤2

评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度8:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估长度9:
Correct: 20.00%, error_type
mismatch    80.0
            20.0
``` 
    
    
    整体效果下降, 步骤2 开始效果劣化. 分析步骤1 的错误case: 

  - 原子交换+1+1
  - 部分轮换+1+1+1+1+1+1+1+1
  - 少原子+1
  - 不能简单变换+2

错误从原子交换 又转向 部分轮换. 再次调整 "子段轮换的增强数据-v3": 

  - 每种子段长度都覆盖一些, 长度越短case越多
  - 随机位置
  - 为镜像selfies也生成增强数据

# 训练方式10

重新制作 "子段轮换的增强数据-v3": 

  - 每种子段长度都覆盖一些, 长度越短case越多
  - 随机位置
  - 为镜像selfies也生成增强数据

随机生成的分子:

  - 长度3(50个)+长度4 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v3
  - 长度3(25个)+长度4(50个)+长度5 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v3
  - 长度3(12个)+长度4(25个)+长度5(50个)+长度6 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v3
  - 长度3(6个)+长度4(12个)+长度5(25个)+长度6(50个)+长度7 (100个), 带等量的镜像增强数据, 带等量的子段轮换的增强数据-v3

commit: 51ce1095f704eb15c7fa6345ea50a26bd584b6cf

训练效果: 

```
# 步骤1

评估长度7:
Correct: 100.00%, error_type
    100.0

评估长度8:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估长度9:
Correct: 45.00%, error_type
mismatch    55.0
            45.0
 
# 步骤2

评估长度7:
Correct: 100.00%, error_type
    100.0

评估长度8:
Correct: 100.00%, error_type
    100.0

评估长度9:
Correct: 50.00%, error_type
mismatch    50.0
            50.0
``` 

评估步骤1,

长度8的错误分布: 一个3-9轮换, 两个原子交换, 一个镜像缺原子

长度9的错误分布: [length_9.analyzed.0113.log](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/length_9.analyzed.0113.log) :

```
复杂变化

4-7轮换
5-8轮换
镜像, 2-7轮换
镜像, 5-8轮换
5-9轮换

镜像, 原子交换, 缺原子
缺原子
缺原子
镜像, 缺原子

原子交换
``` 

发现一个问题: 本次训练的增强数据是原数据的两倍, 但在训练脚本中强制指定为了一倍, 导致如上结果不真实

# 训练方式11

重新制作 "子段轮换的增强数据-v4": 

  - 子段轮换的增强数据-v4 是正常数据的4倍

随机生成的分子:

  - 长度3(50个)+长度4 (100个), 带等量的镜像增强数据, 带4倍的子段轮换的增强数据-v4
  - 长度3(25个)+长度4(50个)+长度5 (100个), 带等量的镜像增强数据, 带4倍的子段轮换的增强数据-v4
  - 长度3(12个)+长度4(25个)+长度5(50个)+长度6 (100个), 带等量的镜像增强数据, 带4倍的子段轮换的增强数据-v4
  - 长度3(6个)+长度4(12个)+长度5(25个)+长度6(50个)+长度7 (100个), 带等量的镜像增强数据, 带4倍的子段轮换的增强数据-v4

数据量: 

```
 /opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/mirror_selfies_from_random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_7.train.json:260
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:260
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_6.train.json:140
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:140
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_5.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_4.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/raw/length_3.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:17
``` 

训练效果: 

```
# 步骤1

评估长度7:
Correct: 100.00%, error_type
    100.0

评估长度8:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估长度9:
Correct: 60.00%, error_type
            60.0
mismatch    40.0

# 步骤2

评估长度7:
Correct: 100.00%, error_type
    100.0

评估长度8:
Correct: 100.00%, error_type
    100.0

评估长度9:
Correct: 75.00%, error_type
            75.0
mismatch    25.0

# 步骤3

评估长度7:
Correct: 100.00%, error_type
    100.0

评估长度8:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估长度9:
Correct: 50.00%, error_type
mismatch    50.0
            50.0 
``` 

长度9的正确率提高到75%, 分析SOTA的长度9的错误偏差: [length_9.analyzed.1059.log](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/length_9.analyzed.1059.log) :

```
镜像, 6-9轮换
6-7交换, 缺原子
缺原子
缺原子
镜像, 缺原子
``` 

训练脚本保存: [run_tasks.random_selfies_only_basic_atom.sh](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/run_tasks.random_selfies_only_basic_atom.sh)

# 训练方式12

另一种尝试: 将"子段轮换的增强数据-v4"作为一个单独的阶段

随机生成的分子:

  - 长度3(50个)+长度4 (100个), 带等量的镜像增强数据
  - 长度3(25个)+长度4(50个)+长度5 (100个), 带等量的镜像增强数据
  - 长度3(12个)+长度4(25个)+长度5(50个)+长度6 (100个), 带等量的镜像增强数据
  - 长度3(6个)+长度4(12个)+长度5(25个)+长度6(50个)+长度7 (100个), 带等量的镜像增强数据
  - 长度3-7的子段轮换的增强数据-v4, 反复训练

长度9的正确率最高是 50%, 作为一个单独的训练阶段效果并不好

# 训练方式13

将现有的SOTA: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v199-20250202-031015/checkpoint-1632 合并进模型, 然后用ring数据进行下一步微调.

先确立基线, 在现有的SOTA上, 再次评估 length8/length9/ring8 的效果: 

```
# 长度7
Correct: 100.00%, error_type
    100.0

# 长度8
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

# 长度9
Correct: 75.00%, error_type
            75.0
mismatch    25.0

# ring数据, 长度7
Correct: 0.00%, error_type
mismatch    100.0

# ring数据, 长度8
Correct: 0.00%, error_type
mismatch    100.0
``` 

合并lora: 

```
cp -R /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v199-20250202-031015/checkpoint-1632 random_selfies_only_basic_atom.length_9.75p.checkpoint
CUDA_VISIBLE_DEVICES=0 swift export --ckpt_dir "/opt/huangyan/mol-ai/models/random_selfies_only_basic_atom.length_9.75p.checkpoint" --merge_lora true
``` 

在合并后的模型上继续训练, 使用ring相关的训练数据, 会发现评估是的输出都套用了json格式符, 比如: 

````
```json\n{\n  "initial_selfies": "[C][N1][N1][N1][N1]",\n  "validation": "The bonding rules are satisfied.",\n  "final_selfies": "[C][N1][N1][N1][N1]"\n}\n```
```` 

也会出现极长selfies表达式, 导致输出不完, 比如: 

````
```json\n{\n  "initial_selfies": "C1=O12C2=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=NC1=
```` 

具体原因尚不清楚 (与之前一次的测试结果类似). 先放弃这条路, 而转由从微调好的基础checkpoint上继续训练.

# 训练方式14

commit: 30060bb0f4cb01303700091c5d815eacff5c0364

代码注意: 

需要使用 --resume_only_model, 让模型忽略之前LR. 但同时需要重置steps.

这里有一个问题, 之前的课程训练, 并没有指定 --resume_only_model, 所以LR在继续训练时, 就已经是衰减后的结果了.

需要安排一个实验, 使用 --resume_only_model, 再次进行课程训练

在之前的SOTA model之上, 开始训练Ring数据(每步骤epoch为3):

  - ring_selfies: 长度5(35个), 长度6(70个)
  - ring_selfies: 长度5(17个), 长度6(35个), 长度7(70个)

训练效果: 

```
# 步骤1
 
评估纯原子数据, 长度7:
Correct: 42.11%, error_type
mismatch    57.894737
            42.105263

评估纯原子数据, 长度8:
Correct: 31.58%, error_type
mismatch    68.421053
            31.578947

评估纯原子数据, 长度9:
Correct: 5.00%, error_type
mismatch    95.0
             5.0

评估ring数据, 长度7:
Correct: 52.50%, error_type
            52.5
mismatch    47.5

评估ring数据, 长度8:
Correct: 17.50%, error_type
mismatch    82.5
            17.5

# 步骤2

评估纯原子数据, 长度7:
Correct: 5.26%, error_type
mismatch    94.736842
             5.263158

评估纯原子数据, 长度8:
Correct: 0.00%, error_type
mismatch    100.0

评估纯原子数据, 长度9:
Correct: 0.00%, error_type
mismatch    100.0

评估ring数据, 长度7:
Correct: 57.50%, error_type
            57.5
mismatch    42.5

评估ring数据, 长度8:
Correct: 12.50%, error_type
mismatch    87.5
            12.5
``` 

在只训练ring数据的情况下, 模型会遗忘普通分子

# 训练方式15

训练ring, 掺入部分普通分子(1/4). 主要步骤:

  - ring_selfies: 长度5(35个), 长度6(70个), 掺入部分普通分子
  - ring_selfies: 长度5(17个), 长度6(35个), 长度7(70个), 掺入部分普通分子

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:2
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:2
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:1
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:1

``` 

训练效果: 

```
# 步骤1
 
评估纯原子数据, 长度7:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估纯原子数据, 长度8:
Correct: 52.63%, error_type
            52.631579
mismatch    47.368421

评估纯原子数据, 长度9:
Correct: 10.00%, error_type
mismatch    90.0
            10.0

评估ring数据, 长度7:
Correct: 62.50%, error_type
            62.5
mismatch    37.5

评估ring数据, 长度8:
Correct: 5.00%, error_type
mismatch    95.0
             5.0

# 步骤2

评估纯原子数据, 长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估纯原子数据, 长度8:
Correct: 42.11%, error_type
mismatch    57.894737
            42.105263

评估纯原子数据, 长度9:
Correct: 10.00%, error_type
mismatch    90.0
            10.0

评估ring数据, 长度7:
Correct: 60.00%, error_type
            60.0
mismatch    40.0

评估ring数据, 长度8:
Correct: 12.50%, error_type
mismatch    87.5
            12.5 
``` 

保持少量普通分子, 可以提高长度7的识别率, 但推理能力在下降

# 训练方式16

训练ring, 增多 掺入的部分普通分子 (1/2). 主要步骤:

  - ring_selfies: 长度5(35个), 长度6(70个), 增多 掺入的部分普通分子
  - ring_selfies: 长度5(17个), 长度6(35个), 长度7(70个), 增多 掺入的部分普通分子

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
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
```
```
# 步骤1
 
评估纯原子数据, 长度7:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估纯原子数据, 长度8:
Correct: 42.11%, error_type
mismatch    57.894737
            42.105263

评估纯原子数据, 长度9:
Correct: 5.00%, error_type
mismatch    95.0
             5.0

评估ring数据, 长度7:
Correct: 50.00%, error_type
mismatch    50.0
            50.0

评估ring数据, 长度8:
Correct: 15.00%, error_type
mismatch    85.0
            15.0

# 步骤2

评估纯原子数据, 长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估纯原子数据, 长度8:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789

评估纯原子数据, 长度9:
Correct: 15.00%, error_type
mismatch    85.0
            15.0

评估ring数据, 长度7:
Correct: 57.50%, error_type
            57.5
mismatch    42.5

评估ring数据, 长度8:
Correct: 22.50%, error_type
mismatch    77.5
            22.5

# 步骤3

评估纯原子数据, 长度7:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估纯原子数据, 长度8:
Correct: 47.37%, error_type
mismatch    52.631579
            47.368421

评估纯原子数据, 长度9:
Correct: 10.00%, error_type
mismatch    90.0
            10.0

评估ring数据, 长度7:
Correct: 62.50%, error_type
            62.5
mismatch    37.5

评估ring数据, 长度8:
Correct: 10.00%, error_type
mismatch    90.0
            10.0
``` 

SOTA在步骤2, 提高了普通分子的数据量后, 对于普通分子长度8的推理能力大幅增强, 但普通分子长度9的推理能力仍比较弱.

对ring数据的长度8的推理能力也有所增强.

# 训练方式17

训练ring, 继续增多 掺入的部分普通分子 (1/1). 主要步骤:

  - ring_selfies: 长度5(35个), 长度6(70个), 继续增多 掺入的部分普通分子
  - ring_selfies: 长度5(17个), 长度6(35个), 长度7(70个), 继续增多 掺入的部分普通分子

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4 
``` 

训练效果: 

```
# 步骤1
 
评估纯原子数据, 长度7:
Correct: 100.00%, error_type
    100.0

评估纯原子数据, 长度8:
Correct: 47.37%, error_type
mismatch    52.631579
            47.368421

评估纯原子数据, 长度9:
Correct: 30.00%, error_type
mismatch    70.0
            30.0

评估ring数据, 长度7:
Correct: 72.50%, error_type
            72.5
mismatch    27.5

评估ring数据, 长度8:
Correct: 10.00%, error_type
mismatch    90.0
            10.0

# 步骤2

评估纯原子数据, 长度7:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估纯原子数据, 长度8:
Correct: 42.11%, error_type
mismatch    57.894737
            42.105263

评估纯原子数据, 长度9:
Correct: 15.00%, error_type
mismatch    85.0
            15.0

评估ring数据, 长度7:
Correct: 70.00%, error_type
            70.0
mismatch    30.0

评估ring数据, 长度8:
Correct: 22.50%, error_type
mismatch    77.5
            22.5

# 步骤3

评估纯原子数据, 长度7:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789

评估纯原子数据, 长度8:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789

评估纯原子数据, 长度9:
Correct: 5.00%, error_type
mismatch    95.0
             5.0

评估ring数据, 长度7:
Correct: 62.50%, error_type
            62.5
mismatch    37.5

评估ring数据, 长度8:
Correct: 27.50%, error_type
mismatch    72.5
            27.5

``` 
    
    
    纯原子数据 和 ring数据, 长度7的准确率都上升, 但纯原子数据的长度8的预测能力, 得多训练一些才能体现. 

评估ring数据的长度7的错误分布: [length_7.analyzed.0147.log](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/length_7.analyzed.0147.log)

```
复杂调整
复杂调整

整体轮换
整体轮换

ring和原子轮换
ring和原子交换
ring和原子交换

原子交换
镜像, 原子交换
镜像, 原子交换
原子交换
``` 

# 训练方式18

尝试锁定lora的一些层, 查看遗忘是否会好一点

commit: 31ab2b30cf4f4e3c721d9cdcdd9297568b52fb73

模型结构: 视觉层32个vision block, 语言层一共28层(每层包括self_attn注意力模块和mlp前馈网络):

指定lora层: "model\\.layers\\.[1-2][0-9]\\..*" (训练最后18层)

训练效果: 

```
# 步骤1
 
评估纯原子数据, 长度7:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估纯原子数据, 长度8:
Correct: 47.37%, error_type
mismatch    52.631579
            47.368421

评估纯原子数据, 长度9:
Correct: 10.00%, error_type
mismatch    90.0
            10.0

评估ring数据, 长度7:
Correct: 80.00%, error_type
            80.0
mismatch    20.0

评估ring数据, 长度8:
Correct: 30.00%, error_type
mismatch    70.0
            30.0

# 步骤2
 
评估纯原子数据, 长度7:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估纯原子数据, 长度8:
Correct: 57.89%, error_type
            57.894737
mismatch    42.105263

评估纯原子数据, 长度9:
Correct: 15.00%, error_type
mismatch    85.0
            15.0

评估ring数据, 长度7:
Correct: 75.00%, error_type
            75.0
mismatch    25.0

评估ring数据, 长度8:
Correct: 15.00%, error_type
mismatch    85.0
            15.0

# 步骤3
 
评估纯原子数据, 长度7:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789

评估纯原子数据, 长度8:
Correct: 52.63%, error_type
            52.631579
mismatch    47.368421

评估纯原子数据, 长度9:
Correct: 15.00%, error_type
mismatch    85.0
            15.0

评估ring数据, 长度7:
Correct: 72.50%, error_type
            72.5
mismatch    27.5

评估ring数据, 长度8:
Correct: 25.00%, error_type
mismatch    75.0
            25.0

 
``` 

指定lora层: "model\\.layers\\.[0-9]\\..*" (只训练前10层)

训练效果: 

```
 # 步骤1
 
评估纯原子数据, 长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估纯原子数据, 长度8:
Correct: 63.16%, error_type
            63.157895
mismatch    36.842105

评估纯原子数据, 长度9:
Correct: 20.00%, error_type
mismatch    80.0
            20.0

评估ring数据, 长度7:
Correct: 77.50%, error_type
            77.5
mismatch    22.5

评估ring数据, 长度8:
Correct: 20.00%, error_type
mismatch    80.0
            20.0

# 步骤2
 
评估纯原子数据, 长度7:
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估纯原子数据, 长度8:
Correct: 42.11%, error_type
mismatch    57.894737
            42.105263

评估纯原子数据, 长度9:
Correct: 15.00%, error_type
mismatch    85.0
            15.0

评估ring数据, 长度7:
Correct: 67.50%, error_type
            67.5
mismatch    32.5

评估ring数据, 长度8:
Correct: 25.00%, error_type
mismatch    75.0
            25.0

``` 

指定lora层: "model\\.layers\\.2[0-9]\\..*" (只训练最后8层)

训练效果: 

```
# 步骤1
 
评估纯原子数据, 长度7:
Correct: 68.42%, error_type
            68.421053
mismatch    31.578947

评估纯原子数据, 长度8:
Correct: 52.63%, error_type
            52.631579
mismatch    47.368421

评估纯原子数据, 长度9:
Correct: 25.00%, error_type
mismatch    75.0
            25.0

评估ring数据, 长度7:
Correct: 57.50%, error_type
            57.5
mismatch    42.5

评估ring数据, 长度8:
Correct: 17.50%, error_type
mismatch    82.5
            17.5

# 步骤2
 
评估纯原子数据, 长度7:
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估纯原子数据, 长度8:
Correct: 36.84%, error_type
mismatch    63.157895
            36.842105

评估纯原子数据, 长度9:
Correct: 20.00%, error_type
mismatch    80.0
            20.0

评估ring数据, 长度7:
Correct: 60.00%, error_type
            60.0
mismatch    40.0

评估ring数据, 长度8:
Correct: 27.50%, error_type
mismatch    72.5
            27.5
``` 

并看不出来哪一些层冻结后能明显保持记忆能力或推理能力.

# 训练方式19

基线: 训练方式17

训练ring, 加入基于普通分子的rotate训练数据. 主要步骤:

  - ring_selfies: 长度5(35个), 长度6(70个), 掺入的部分普通分子, 加入基于普通分子的rotate训练数据
  - ring_selfies: 长度5(17个), 长度6(35个), 长度7(70个), 掺入的部分普通分子, 加入基于普通分子的rotate训练数据

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
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
``` 

训练效果: 

```
# 步骤1
 
评估长度7:
Correct: 100.00%, error_type
    100.0
 
评估长度8:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789
 
评估长度9:
Correct: 25.00%, error_type
mismatch    75.0
            25.0

评估ring长度7:
Correct: 65.00%, error_type
            65.0
mismatch    35.0

评估ring长度8:
Correct: 22.50%, error_type
mismatch    77.5
            22.5

# 步骤2
 
评估长度7:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789
 
评估长度8:
Correct: 36.84%, error_type
mismatch    63.157895
            36.842105
 
评估长度9:
Correct: 5.00%, error_type
mismatch    95.0
             5.0

评估ring长度7:
Correct: 67.50%, error_type
            67.5
mismatch    32.5

评估ring长度8:
Correct: 7.50%, error_type
mismatch    92.5
             7.5

# 步骤3
 
评估长度7:
Correct: 100.00%, error_type
    100.0
 
评估长度8:
Correct: 57.89%, error_type
            57.894737
mismatch    42.105263
 
评估长度9:
Correct: 25.00%, error_type
mismatch    75.0
            25.0

评估ring长度7:
Correct: 70.00%, error_type
            70.0
mismatch    30.0

评估ring长度8:
Correct: 20.00%, error_type
mismatch    80.0
            20.0
``` 

ring数据的准确率没有提升, 分析错误: [ring.length_7.analyzed.1233.log](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/ring.length_7.analyzed.1233.log) :

```
复杂变化

镜像, 1-5轮换
镜像, 3-5轮换
镜像, 3-5轮换
镜像, 4-5轮换
镜像, 4-5轮换

4-6轮换
4-7轮换
4-7轮换
5-7轮换
5-7轮换
5-7轮换
2-3轮换
``` 

增加普通原子的轮换增强, 对ring数据没有用. 

# 训练方式20

训练ring, 加入基于ring分子的rotate训练数据. 主要步骤:

  - ring_selfies: 长度5(35个), 长度6(70个), 掺入等量的部分普通分子, 加入等量的基于ring分子的rotate训练数据
  - ring_selfies: 长度5(17个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 加入等量的基于ring分子的rotate训练数据

commit: 6cca2fa499b1dac646ff8f354b521a0af59945a8

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/enhanced_by_image/length_5.train.json:17

``` 

训练效果: 

```
# 步骤1
 
评估长度7:
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789
 
评估长度8:
Correct: 63.16%, error_type
            63.157895
mismatch    36.842105
 
评估长度9:
Correct: 15.00%, error_type
mismatch    85.0
            15.0

评估ring长度7:
Correct: 67.50%, error_type
            67.5
mismatch    32.5

评估ring长度8:
Correct: 22.50%, error_type
mismatch    77.5
            22.5

# 步骤2
 
评估长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158
 
评估长度8:
Correct: 68.42%, error_type
            68.421053
mismatch    31.578947
 
评估长度9:
Correct: 30.00%, error_type
mismatch    70.0
            30.0

评估ring长度7:
Correct: 87.50%, error_type
            87.5
mismatch    12.5

评估ring长度8:
Correct: 35.00%, error_type
mismatch    65.0
            35.0

# 步骤3
 
评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316
 
评估长度8:
Correct: 57.89%, error_type
            57.894737
mismatch    42.105263
 
评估长度9:
Correct: 25.00%, error_type
mismatch    75.0
            25.0

评估ring长度7:
Correct: 85.00%, error_type
            85.0
mismatch    15.0

评估ring长度8:
Correct: 50.00%, error_type
            50.0
mismatch    50.0
``` 

效果: 

  - 长度7的ring分子识别率提高到 87.50% (步骤2)
  - 到步骤3时, 长度7的ring分子识别率不再提高, 但长度8再提高到50%
  - 对于普通分子的识别有所下降, 应该是普通分子的占比在下降

分析长度7的ring分子的错误分布: [length_7.ring.analyzed.2255.log](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/length_7.ring.analyzed.2255.log) :

```
1-7轮换2次
1-7轮换
起点3, 4-6轮换
4-6轮换
3-6轮换
``` 

继续扩大 基于ring分子的rotate训练数据 的数量

# 训练方式21

训练ring, 扩大基于ring分子的rotate训练数据. 主要步骤:

  - ring_selfies: 长度5(35个), 长度6(70个), 掺入等量的部分普通分子, 3倍的基于ring分子的rotate训练数据
  - ring_selfies: 长度5(17个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 3倍的基于ring分子的rotate训练数据

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/raw/length_7.train.json:210
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/enhanced_by_image/length_7.train.json:210
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/raw/length_6.train.json:105
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/enhanced_by_image/length_6.train.json:105
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/raw/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_ring_selfies/enhanced_by_image/length_5.train.json:20 
``` 

训练效果: 

```
 # 步骤1
 
评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316
 
评估长度8:
Correct: 63.16%, error_type
            63.157895
mismatch    36.842105
 
评估长度9:
Correct: 20.00%, error_type
mismatch    80.0
            20.0

评估ring长度7:
Correct: 87.50%, error_type
            87.5
mismatch    12.5

评估ring长度8:
Correct: 22.50%, error_type
mismatch    77.5
            22.5

# 步骤2
 
评估长度7:
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158
 
评估长度8:
Correct: 26.32%, error_type
mismatch    73.684211
            26.315789
 
评估长度9:
Correct: 15.00%, error_type
mismatch    85.0
            15.0

评估ring长度7:
Correct: 90.00%, error_type
            90.0
mismatch    10.0

评估ring长度8:
Correct: 17.50%, error_type
mismatch    82.5
            17.5

# 步骤3
 
评估长度7:
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316
 
评估长度8:
Correct: 57.89%, error_type
            57.894737
mismatch    42.105263
 
评估长度9:
Correct: 5.00%, error_type
mismatch    95.0
             5.0

评估ring长度7:
Correct: 95.00%, error_type
            95.0
mismatch     5.0

评估ring长度8:
Correct: 27.50%, error_type
mismatch    72.5
            27.5
``` 

增加rotate增强数据, 可以提高ring的准确度到95%,

步骤3的错误都是2-3轮换,

步骤2的错误: 

```
37   /opt/huangyan/mol-ai/images/ring_selfies/length_7/22.png            {
"selfies": "[C][C][N][N][Ring1][Ring2][C]"}                  C1CNN1C    {
"selfies": "[C][C][C][N][N][Ring1][Ring2]"}   CC1CNN1    0.157895   
mismatch
3-7轮换

36   /opt/huangyan/mol-ai/images/ring_selfies/length_7/70.png            {
"selfies": "[N][O][N][Ring1][Ring1][N][N]"}                  N1ON1NN    {
"selfies": "[N][N][O][N][Ring1][Ring1][N]"}   NN1ON1N    0.176471   
mismatch
2-7轮换

35    /opt/huangyan/mol-ai/images/ring_selfies/length_7/5.png            {
"selfies": "[C][C][N][N][C][Ring1][Ring1]"}                  CCN1NC1    {
"selfies": "[C][N][C][N][Ring1][Ring1][C]"}   CN1CN1C    0.176471   
mismatch
多次轮换

25  /opt/huangyan/mol-ai/images/ring_selfies/length_7/167.png            {
"selfies": "[C][C][N][C][O][Ring1][Ring2]"}                  CC1NCO1    {
"selfies": "[C][C][C][N][O][Ring1][Ring2]"}   CC1CNO1    0.333333   
mismatch
3-4轮换
``` 

步骤2, 既保持了长度7的记忆, 又在ring上有改进, 使用这一版模型checkpoint, 来训练branch数据: 

ring的checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v466-20250205-040035/checkpoint-219

(基础原子的checkpoint: /opt/huangyan/mol-ai/output/qwen2-vl-7b-instruct/v199-20250202-031015/checkpoint-1632)

# 训练方式21

训练branch, 主要步骤:

  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个)
  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个)

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16
``` 

训练效果: 

```
# 步骤1
 
评估普通分子, 长度7
Correct: 0.00%, error_type
mismatch    100.0

评估普通分子, 长度8
Correct: 0.00%, error_type
mismatch    100.0

评估ring分子, 长度7
Correct: 0.00%, error_type
mismatch    100.0

评估ring分子, 长度8 
Correct: 0.00%, error_type
mismatch    100.0

评估branch分子, 长度7
Correct: 26.67%, error_type
mismatch    73.333333
            26.666667

评估branch分子, 长度8
Correct: 3.33%, error_type
mismatch    96.666667
             3.333333
 
# 步骤2

评估普通分子, 长度7
Correct: 0.00%, error_type
mismatch    100.0

评估普通分子, 长度8

Correct: 0.00%, error_type
mismatch    100.0

评估ring分子, 长度7
Correct: 0.00%, error_type
mismatch    100.0

评估ring分子, 长度8 
Correct: 0.00%, error_type
mismatch    100.0

评估branch分子, 长度7
Correct: 40.00%, error_type
mismatch    60.0
            40.0

评估branch分子, 长度8
Correct: 10.00%, error_type
mismatch    90.0
            10.0
``` 

# 训练方式22

训练branch, 掺入等量的部分普通分子, 主要步骤:

  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个), 掺入等量的部分普通分子
  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16

``` 

训练效果: 

```
# 步骤1
 
评估普通分子, 长度7
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估普通分子, 长度8
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估ring分子, 长度7
Correct: 0.00%, error_type
mismatch    100.0

评估ring分子, 长度8 
Correct: 0.00%, error_type
mismatch    100.0

评估branch分子, 长度7
Correct: 66.67%, error_type
            66.666667
mismatch    33.333333

评估branch分子, 长度8
Correct: 50.00%, error_type
mismatch    50.0
            50.0
 
# 步骤2

评估普通分子, 长度7
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估普通分子, 长度8
Correct: 68.42%, error_type
            68.421053
mismatch    31.578947

评估ring分子, 长度7
Correct: 0.00%, error_type
mismatch    100.0

评估ring分子, 长度8 
Correct: 0.00%, error_type
mismatch    100.0

评估branch分子, 长度7
Correct: 66.67%, error_type
            66.666667
mismatch    33.333333

评估branch分子, 长度8
Correct: 56.67%, error_type
            56.666667
mismatch    43.333333
 
# 步骤3

评估普通分子, 长度7
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估普通分子, 长度8
Correct: 57.89%, error_type
            57.894737
mismatch    42.105263

评估ring分子, 长度7
Correct: 0.00%, error_type
mismatch    100.0

评估ring分子, 长度8 
Correct: 0.00%, error_type
mismatch    100.0

评估branch分子, 长度7
Correct: 70.00%, error_type
            70.0
mismatch    30.0

评估branch分子, 长度8
Correct: 46.67%, error_type
mismatch    53.333333
            46.666667
``` 

回忆 普通分子, 能增强对branch分子的理解

# 训练方式23

训练branch, 掺入等量的ring分子, 主要步骤:

  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个), 掺入等量的部分普通分子, 掺入等量的ring分子
  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 掺入等量的ring分子

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16
``` 

训练效果: 

```
 # 步骤1
 
评估普通分子, 长度7
Correct: 73.68%, error_type
            73.684211
mismatch    26.315789

评估普通分子, 长度8
Correct: 36.84%, error_type
mismatch    63.157895
            36.842105

评估ring分子, 长度7
Correct: 90.00%, error_type
            90.0
mismatch    10.0

评估ring分子, 长度8 
Correct: 17.50%, error_type
mismatch    82.5
            17.5

评估branch分子, 长度7
Correct: 73.33%, error_type
            73.333333
mismatch    26.666667

评估branch分子, 长度8
Correct: 43.33%, error_type
mismatch    56.666667
            43.333333
 
# 步骤2

评估普通分子, 长度7
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估普通分子, 长度8
Correct: 15.79%, error_type
mismatch    84.210526
            15.789474

评估ring分子, 长度7
Correct: 100.00%, error_type
    100.0

评估ring分子, 长度8 
Correct: 22.50%, error_type
mismatch    77.5
            22.5

评估branch分子, 长度7
Correct: 73.33%, error_type
            73.333333
mismatch    26.666667

评估branch分子, 长度8
Correct: 46.67%, error_type
mismatch    53.333333
            46.666667
 
# 步骤3

评估普通分子, 长度7
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估普通分子, 长度8
Correct: 42.11%, error_type
mismatch    57.894737
            42.105263

评估ring分子, 长度7
Correct: 97.50%, error_type
            97.5
mismatch     2.5

评估ring分子, 长度8 
Correct: 17.50%, error_type
mismatch    82.5
            17.5

评估branch分子, 长度7
Correct: 73.33%, error_type
            73.333333
mismatch    26.666667

评估branch分子, 长度8
Correct: 43.33%, error_type
mismatch    56.666667
            43.333333
``` 

  - 回忆 ring 分子, 看起来比较"容易"
  - 回忆 ring 分子, 对普通分子的记忆有"压制", 其需要一段时间才能回忆起来
  - 回忆 ring 分子, 对branch分子的理解没有改善

降低ring分子的配比试试

# 对于branch分子的发现

使用以下代码, 对同一个branch分子进行 selfies库验证, 和rdkit库验证

```
import selfies as sf
selfies = "[O][C][Branch1][Ring1][C][N][N]"
smiles = sf.decoder(selfies)
reencoded_selfies = sf.encoder(smiles)
print(reencoded_selfies)
print(smiles)

from rdkit import Chem
true_response_mol = Chem.MolFromSmiles(smiles)
new_smiles = Chem.MolToSmiles(true_response_mol, rootedAtAtom=int(0))
new_selfies = sf.encoder(new_smiles)
print(new_selfies)
print(new_smiles)
``` 

其结果: 两种方式重写出的表达式会不同, 主链和支链会对调

```
[O][C][Branch1][Ring1][C][N][N]
OC(CN)N
[O][C][Branch1][C][N][C][N]
OC(N)CN
``` 

# 训练方式24

训练branch, 掺入1/2的ring分子, 主要步骤:

  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个), 掺入等量的部分普通分子, 掺入1/2的ring分子
  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 掺入1/2的ring分子

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16 
``` 

训练效果: 

```
 # 步骤1
 
评估普通分子, 长度7
Correct: 100.00%, error_type
    100.0

评估普通分子, 长度8
Correct: 57.89%, error_type
            57.894737
mismatch    42.105263

评估ring分子, 长度7
Correct: 92.50%, error_type
            92.5
mismatch     7.5

评估ring分子, 长度8 
Correct: 12.50%, error_type
mismatch    87.5
            12.5

评估branch分子, 长度7
Correct: 63.33%, error_type
            63.333333
mismatch    36.666667

评估branch分子, 长度8
Correct: 46.67%, error_type
mismatch    53.333333
            46.666667
 
# 步骤2

评估普通分子, 长度7
Correct: 68.42%, error_type
            68.421053
mismatch    31.578947

评估普通分子, 长度8
Correct: 47.37%, error_type
mismatch    52.631579
            47.368421

评估ring分子, 长度7
Correct: 85.00%, error_type
            85.0
mismatch    15.0

评估ring分子, 长度8 
Correct: 17.50%, error_type
mismatch    82.5
            17.5

评估branch分子, 长度7
Correct: 63.33%, error_type
            63.333333
mismatch    36.666667

评估branch分子, 长度8
Correct: 46.67%, error_type
mismatch    53.333333
            46.666667
 
# 步骤3

评估普通分子, 长度7
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估普通分子, 长度8
Correct: 36.84%, error_type
mismatch    63.157895
            36.842105

评估ring分子, 长度7
Correct: 87.50%, error_type
            87.5
mismatch    12.5

评估ring分子, 长度8 
Correct: 12.50%, error_type
mismatch    87.5
            12.5

评估branch分子, 长度7
Correct: 73.33%, error_type
            73.333333
mismatch    26.666667

评估branch分子, 长度8
Correct: 40.00%, error_type
mismatch    60.0
            40.0
``` 

降低ring样例后, 记忆仍能保持, 但branch分子的识别率在同等情况下有下降. 

评估SOTA的branch分子识别错误: [length_7.branch.analyzed.1646.log](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/length_7.branch.analyzed.1646.log) :

```
镜像, 1-7轮换
少原子, Chem重写后5-7轮换
丢失branch
1-7轮换
1和4对调
4-5轮换
branch长度错误, 5-7轮换
1和4对调
``` 

增加对于branch分子的rotate增强

# 训练方式25

训练branch, 掺入等量的branch分子的rotate增强, 主要步骤:

  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个), 掺入等量的部分普通分子, 掺入1/2的ring分子, 掺入等量的branch分子的rotate增强
  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 掺入1/2的ring分子, 掺入等量的branch分子的rotate增强

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_5.train.json:17

``` 

训练效果: 

```
 # 步骤1
 
评估普通分子, 长度7
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估普通分子, 长度8
Correct: 0.00%, error_type
mismatch    100.0

评估ring分子, 长度7
Correct: 75.00%, error_type
            75.0
mismatch    25.0

评估ring分子, 长度8 
Correct: 17.50%, error_type
mismatch    80.0
            20.0

评估branch分子, 长度7
Correct: 73.33%, error_type
            73.333333
mismatch    26.666667

评估branch分子, 长度8
Correct: 33.33%, error_type
mismatch    66.666667
            33.333333

# 步骤2
 
评估普通分子, 长度7
Correct: 84.21%, error_type
            84.210526
mismatch    15.789474

评估普通分子, 长度8
Correct: 31.58%, error_type
mismatch    68.421053
            31.578947

评估ring分子, 长度7
Correct: 75.00%, error_type
            75.0
mismatch    25.0

评估ring分子, 长度8 
Correct: 17.50%, error_type
mismatch    82.5
            17.5

评估branch分子, 长度7
Correct: 70.00%, error_type
            70.0
mismatch    30.0

评估branch分子, 长度8
Correct: 40.00%, error_type
mismatch    60.0
            40.0

# 步骤3
 
评估普通分子, 长度7
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估普通分子, 长度8
Correct: 47.37%, error_type
mismatch    52.631579
            47.368421

评估ring分子, 长度7
Correct: 85.00%, error_type
            85.0
mismatch    15.0

评估ring分子, 长度8 
Correct: 20.00%, error_type
mismatch    80.0
            20.0

评估branch分子, 长度7
Correct: 80.00%, error_type
            80.0
mismatch    20.0

评估branch分子, 长度8
Correct: 46.67%, error_type
mismatch    53.333333
            46.666667
``` 

对branch的识别率提升到80%, 没有预期的好. 分析SOTA的branch分子识别错误: [length_7.branch.analyzed.1757.log](/assets/01KJBZR0H7TBJT1Y07GE1DHEBD/length_7.branch.analyzed.1757.log) :

```
5-6轮换
4-6轮换
起点3, 1/6对调
Chem重写后5-6轮换
Chem重写后4-7轮换
3-4轮换
``` 

看起来继续增加轮换数据可能是有效的

# 训练方式26

训练branch, 掺入等量的branch分子的rotate增强, 主要步骤:

  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个), 掺入等量的部分普通分子, 掺入1/2的ring分子, 掺入2倍的branch分子的rotate增强
  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 掺入1/2的ring分子, 掺入2倍的branch分子的rotate增强

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_7.train.json:140
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_7.train.json:140
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_6.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_6.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_5.train.json:35

``` 

训练效果: 

```
 # 步骤1
 
评估普通分子, 长度7
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估普通分子, 长度8
Correct: 26.32%, error_type
mismatch    73.684211
            26.315789

评估ring分子, 长度7
Correct: 85.00%, error_type
            85.0
mismatch    15.0

评估ring分子, 长度8 
Correct: 17.50%, error_type
mismatch    82.5
            17.5

评估branch分子, 长度7
Correct: 76.67%, error_type
            76.666667
mismatch    23.333333

评估branch分子, 长度8
Correct: 40.00%, error_type
mismatch    60.0
            40.0

# 步骤2
 
评估普通分子, 长度7
Correct: 100.00%, error_type
    100.0

评估普通分子, 长度8
Correct: 42.11%, error_type
mismatch    57.894737
            42.105263

评估ring分子, 长度7
Correct: 85.00%, error_type
            85.0
mismatch    15.0

评估ring分子, 长度8 
Correct: 20.00%, error_type
mismatch    80.0
            20.0

评估branch分子, 长度7
Correct: 80.00%, error_type
            80.0
mismatch    20.0

评估branch分子, 长度8
Correct: 43.33%, error_type
mismatch    56.666667
            43.333333

# 步骤3
 
评估普通分子, 长度7
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估普通分子, 长度8
Correct: 47.37%, error_type
mismatch    52.631579
            47.368421

评估ring分子, 长度7
Correct: 80.00%, error_type
            80.0
mismatch    20.0

评估ring分子, 长度8 
Correct: 12.50%, error_type
mismatch    87.5
            12.5

评估branch分子, 长度7
Correct: 86.67%, error_type
            86.666667
mismatch    13.333333

评估branch分子, 长度8
Correct: 56.67%, error_type
            56.666667
mismatch    43.333333
``` 

branch分子的识别率提高到 86%

将ring分子调整成等量, 训练效果: 

```
# 步骤1
 
评估普通分子, 长度7
Correct: 94.74%, error_type
            94.736842
mismatch     5.263158

评估普通分子, 长度8
Correct: 47.37%, error_type
mismatch    52.631579
            47.368421

评估ring分子, 长度7
Correct: 92.50%, error_type
            92.5
mismatch     7.5

评估ring分子, 长度8 
Correct: 22.50%, error_type
mismatch    77.5
            22.5

评估branch分子, 长度7
Correct: 70.00%, error_type
            70.0
mismatch    30.0

评估branch分子, 长度8
Correct: 50.00%, error_type
            50.0
mismatch    50.0

# 步骤2
 
评估普通分子, 长度7
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估普通分子, 长度8
Correct: 68.42%, error_type
            68.421053
mismatch    31.578947

评估ring分子, 长度7
Correct: 82.50%, error_type
            82.5
mismatch    17.5

评估ring分子, 长度8 
Correct: 22.50%, error_type
mismatch    77.5
            22.5

评估branch分子, 长度7
Correct: 93.33%, error_type
            93.333333
mismatch     6.666667

评估branch分子, 长度8
Correct: 46.67%, error_type
mismatch    53.333333
            46.666667

# 步骤3
 
评估普通分子, 长度7
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估普通分子, 长度8
Correct: 42.11%, error_type
mismatch    57.894737
            42.105263

评估ring分子, 长度7
Correct: 80.00%, error_type
            80.0
mismatch    20.0

评估ring分子, 长度8 
Correct: 17.50%, error_type
mismatch    82.5
            17.5

评估branch分子, 长度7
Correct: 86.67%, error_type
            86.666667
mismatch    13.333333

评估branch分子, 长度8
Correct: 43.33%, error_type
mismatch    56.666667
            43.333333
``` 

branch的识别率提高到93%, 但对ring的记忆力在下降??

将 branch分子的rotate增强 也调整成等量, 训练效果: 

```
 # 步骤1
 
评估普通分子, 长度7
Correct: 78.95%, error_type
            78.947368
mismatch    21.052632

评估普通分子, 长度8
Correct: 36.84%, error_type
mismatch    63.157895
            36.842105

评估ring分子, 长度7
Correct: 87.50%, error_type
            87.5
mismatch    12.5

评估ring分子, 长度8 
Correct: 22.50%, error_type
mismatch    77.5
            22.5

评估branch分子, 长度7
Correct: 63.33%, error_type
            63.333333
mismatch    36.666667

评估branch分子, 长度8
Correct: 43.33%, error_type
mismatch    56.666667
            43.333333

# 步骤2
 
评估普通分子, 长度7
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估普通分子, 长度8
Correct: 36.84%, error_type
mismatch    63.157895
            36.842105

评估ring分子, 长度7
Correct: 92.50%, error_type
            92.5
mismatch     7.5

评估ring分子, 长度8 
Correct: 15.00%, error_type
mismatch    85.0
            15.0

评估branch分子, 长度7
Correct: 76.67%, error_type
            80.0
mismatch    20.0

评估branch分子, 长度8
Correct: 43.33%, error_type
mismatch    56.666667
            43.333333

# 步骤3
 
评估普通分子, 长度7
Correct: 89.47%, error_type
            89.473684
mismatch    10.526316

评估普通分子, 长度8
Correct: 31.58%, error_type
mismatch    68.421053
            31.578947

评估ring分子, 长度7
Correct: 85.00%, error_type
            85.0
mismatch    15.0

评估ring分子, 长度8 
Correct: 22.50%, error_type
mismatch    77.5
            22.5

评估branch分子, 长度7
Correct: 76.67%, error_type
            76.666667
mismatch    23.333333

评估branch分子, 长度8
Correct: 56.67%, error_type
            56.666667
mismatch    43.333333

``` 

branch的识别率下降

# 训练方式27

训练branch, 两阶段使用不同的LR, 主要步骤:

  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个), 掺入等量的部分普通分子, 掺入等量的ring分子, 掺入2倍的branch分子的rotate增强
  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 掺入等量的ring分子, 掺入2倍的branch分子的rotate增强

| LR=4e-4, 2e-4 | LR=2e-4, 1e-4 | LR=1e-4, 5e-5 |  |
| --- | --- | --- | --- |
      
    
    步骤=1, 评估普通分子, 长度7

| 89.47%| 94.74%| 94.74%  
      
    
    步骤=1, 评估ring分子, 长度7

| 95.00%| 95.00%| 97.50%  
      
    
    步骤=1, 评估branch分子

| 86.67%| 73.33%| 83.33%  
      
    
    步骤=2, 评估普通分子, 长度7

| 84.21%| 100.00%| 94.74%  
      
    
    步骤=2, 评估ring分子, 长度7

| 90.00%| 92.50%| 90.00%  
      
    
    步骤=2, 评估branch分子

| 80.00%| 83.33%| 76.67%  
      
    
    步骤=3, 评估普通分子, 长度7

| 100.00%| 84.21%| 89.47%  
      
    
    步骤=3, 评估ring分子, 长度7

| 92.50%| 100.00%| 87.50%  
      
    
    步骤=3, 评估branch分子

| 80.00%| 83.33%| 76.67%  
  
LR降低, 有利于保持记忆. 对branch分子的评估准确率没有提高.

分析对branch分子的评估错误 (标红项, 对普通分子和ring分子的记忆保持最好):

```
25   /opt/huangyan/mol-ai/images/branch_selfies/length_7/11.png      {
"selfies": "[N][Branch1][C][N][O][N][N]"}          N(N)ONN      {
"selfies": "[N][O][N][Branch1][C][N][N]"}   NON(N)N    0.142857   
mismatch
镜像, 1-7轮换
[N][O][N][Branch1][C][N][N]
[O][Branch1][C][N][N][Branch1][C][N][N]
[N][Branch1][C][N][Branch1][C][N][O][N]
[N][N][Branch1][C][N][O][N]
[N][N][Branch1][C][N][O][N]

1   /opt/huangyan/mol-ai/images/branch_selfies/length_7/125.png                  {
"selfies": "[C][C][C][N][N]"}            CCCNN  {
"selfies": "[N][C][Branch1][Ring1][C][C][N]"}   NC(CC)N    0.235294   
mismatch
起点4, 缺少branch结构
[N][C][Branch1][C][N][C][C]
[C][Branch1][C][N][Branch1][C][N][C][C]
[C][Branch1][C][C][C][Branch1][C][N][N]
[C][C][C][Branch1][C][N][N]
[N][C][Branch1][C][N][C][C]

21   /opt/huangyan/mol-ai/images/branch_selfies/length_7/81.png  {
"selfies": "[C][Branch1][Ring1][N][O][O][N]"}          C(NO)ON      {
"selfies": "[O][Branch1][C][O][N][C][N]"}   O(O)NCN    0.238095   
mismatch
偏差: [N][C][N][O][O]和[N][O][C][N][O]的branch形式
[O][Branch1][C][O][N][C][N]
[O][O][N][C][N]
[N][Branch1][Ring1][C][N][O][O]
[C][Branch1][C][N][N][O][O]
[N][C][N][O][O]

29    /opt/huangyan/mol-ai/images/branch_selfies/length_7/3.png                  {
"selfies": "[O][C][C][C][C]"}            OCCCC      {
"selfies": "[O][C][C][Branch1][C][C][C]"}   OCC(C)C    0.250000   
mismatch
缺少branch结构
[O][C][C][Branch1][C][C][C]
[C][Branch1][C][O][C][Branch1][C][C][C]
[C][Branch1][C][C][Branch1][C][C][C][O]
[C][C][Branch1][C][C][C][O]
[C][C][Branch1][C][C][C][O]

13   /opt/huangyan/mol-ai/images/branch_selfies/length_7/51.png      {
"selfies": "[O][C][Branch1][C][O][O][N]"}          OC(O)ON      {
"selfies": "[N][C][Branch1][C][O][O][O]"}   NC(O)OO    0.357143   
mismatch
1,7对调
起点3, 5-7轮换(分支内外轮换)
[N][C][Branch1][C][O][O][O]
[C][Branch1][C][N][Branch1][C][O][O][O]
[O][C][Branch1][C][N][O][O]
[O][Branch1][C][O][C][Branch1][C][N][O]
[O][O][C][Branch1][C][N][O]
 
------
 
7    /opt/huangyan/mol-ai/images/branch_selfies/length_7/70.png  {
"selfies": "[C][Branch1][Ring1][C][O][N][N]"}          C(CO)NN      {
"selfies": "[O][Branch1][C][N][C][C][N]"}   O(N)CCN    0.095238   
mismatch
起点3, 5-6轮换(分支内外轮换)
[O][Branch1][C][N][C][C][N]
[N][O][C][C][N]
[C][Branch1][Ring1][C][N][O][N]
[C][Branch1][C][N][C][O][N]
[N][C][C][O][N]

10  /opt/huangyan/mol-ai/images/branch_selfies/length_7/100.png  {
"selfies": "[C][Branch1][Ring2][O][C][N][N]"}          C(OCN)N  {
"selfies": "[O][Branch1][Ring2][C][N][N][C]"}   O(CNN)C    0.166667   
mismatch
差别: [C][O][C][N][N]和[N][C][O][C][N]的branch形式
[O][Branch1][C][C][C][N][N]
[C][Branch1][Ring1][N][N][O][C]
[N][Branch1][C][N][C][O][C]
[N][N][C][O][C]
[C][O][C][N][N]

11   /opt/huangyan/mol-ai/images/branch_selfies/length_7/40.png  {
"selfies": "[C][Branch1][Ring2][C][N][C][O]"}          C(CNC)O      {
"selfies": "[N][Branch1][C][O][C][C][C]"}   N(O)CCC    0.263158   
mismatch
差别: [C][C][C][N][O]和[C][N][C][C][O]的branch形式
[N][Branch1][C][O][C][C][C]
[O][N][C][C][C]
[C][Branch1][Ring1][C][C][N][O]
[C][Branch1][C][C][C][N][O]
[C][C][C][N][O]

29    /opt/huangyan/mol-ai/images/branch_selfies/length_7/3.png      {
"selfies": "[O][C][Branch1][C][C][C][C]"}          OC(C)CC      {
"selfies": "[O][C][C][Branch1][C][C][C]"}   OCC(C)C    0.357143   
mismatch
3-5轮换(分支起点调整)
[O][C][C][Branch1][C][C][C]
[C][Branch1][C][O][C][Branch1][C][C][C]
[C][Branch1][C][C][Branch1][C][C][C][O]
[C][C][Branch1][C][C][C][O]
[C][C][Branch1][C][C][C][O]

``` 

先解决 "差别: [C][O][C][N][N]和[N][C][O][C][N]的branch形式" 和 "缺少branch结构" 的问题:

使用增强数据: 在长度5的分子中添加[branch]操作符, 让其变成带分支的长度7的分子, 将变化前后的分子都放在训练文件中, 让模型增强对长度5和带ring的长度7之间的关系的理解.

# 训练方式28

训练branch, 掺入branch增强数据(原分子+添加[branch]操作符的分子), 主要步骤:

  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个), 掺入等量的部分普通分子, 掺入等量的ring分子, 掺入2倍的branch分子的rotate增强, LR=4e-4, 掺入branch增强数据(原分子+添加[branch]操作符的分子)
  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 掺入等量的ring分子, 掺入2倍的branch分子的rotate增强, LR=2e-4, 掺入branch增强数据(原分子+添加[branch]操作符的分子)

commit: b08905756136aad80b6c929999586725b7e57768

|  | 掺入等量branch增强数据 | 掺入2倍的branch增强数据 |
| --- | --- | --- |
      
    
    步骤=1, 评估普通分子, 长度7

| 94.74%| 94.74%  
      
    
    步骤=1, 评估ring分子, 长度7

| 85.00%| 85.00%  
      
    
    步骤=1, 评估branch分子

| 80.00%| 90.00%  
      
    
    步骤=2, 评估普通分子, 长度7

| 94.74%| 84.21%  
      
    
    步骤=2, 评估ring分子, 长度7

| 95.00%| 77.50%  
      
    
    步骤=2, 评估branch分子

| 90.00%| 86.67%  
      
    
    步骤=3, 评估普通分子, 长度7

| 100.00%| 73.68%  
      
    
    步骤=3, 评估ring分子, 长度7

| 92.50%| 82.50%  
      
    
    步骤=3, 评估branch分子

| 83.33%| 83.33%  
  
对branch的识别率上升

# 训练方式29

训练branch, 掺入branch增强数据(~~原分子+~~ 添加[branch]操作符的分子), 主要步骤:

  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个), 掺入等量的部分普通分子, 掺入等量的ring分子, 掺入2倍的branch分子的rotate增强, LR=4e-4, 掺入branch增强数据(添加[branch]操作符)
  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 掺入等量的ring分子, 掺入2倍的branch分子的rotate增强, LR=2e-4, 掺入branch增强数据(添加[branch]操作符)

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_7.train.json:140
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_7.train.json:140
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_6.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_6.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/raw/length_7.train.json:49
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:49
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/raw/length_6.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/raw/length_5.train.json:5
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:5

```

|  | 掺入等量branch增强数据 | 掺入2倍的branch增强数据 |
| --- | --- | --- |
      
    
    步骤=1, 评估普通分子, 长度7

| 78.95%| 78.95%  
      
    
    步骤=1, 评估ring分子, 长度7

| 85.00%| 92.50%  
      
    
    步骤=1, 评估branch分子

| 66.67%| 83.33%  
      
    
    步骤=2, 评估普通分子, 长度7

| 73.68%| 84.21%  
      
    
    步骤=2, 评估ring分子, 长度7

| 90.00%| 92.50%  
      
    
    步骤=2, 评估branch分子

| 73.33%| 86.67%  
      
    
    步骤=3, 评估普通分子, 长度7

| 100.00%| 89.47%  
      
    
    步骤=3, 评估ring分子, 长度7

| 90.00%| 87.50%  
      
    
    步骤=3, 评估branch分子

| 76.67%| 86.67%  
  
发现增强数据量不够, 需要增多增强数据, 重做试验29.

由于长度5的分子有限, 增加[branch]操作符生成的长度7的数据只有90个. 所以 "掺入2倍的branch增强数据" 实际上只有90个 (不切分训练集和校验集)

  

数据量: 

```
 /opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_7.train.json:140
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_7.train.json:140
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_6.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_6.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/raw/length_7.json:90
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/enhanced_by_image/length_7.json:90
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/raw/length_6.json:61
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/enhanced_by_image/length_6.json:61
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/raw/length_5.json:26
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/enhanced_by_image/length_5.json:26

``` 

  

|  | 掺入等量branch增强数据 (70个) | 掺入2倍的branch增强数据(只有90个) |
| --- | --- | --- |
      
    
    步骤=1, 评估普通分子, 长度7

| 89.47%| 94.74%  
      
    
    步骤=1, 评估ring分子, 长度7

| 92.50%| 80.00%  
      
    
    步骤=1, 评估branch分子

| 73.33%| 90.00%  
      
    
    步骤=2, 评估普通分子, 长度7

| 94.74%| 84.21%  
      
    
    步骤=2, 评估ring分子, 长度7

| 87.50%| 85.00%  
      
    
    步骤=2, 评估branch分子

| 83.33%| 86.67%  
      
    
    步骤=3, 评估普通分子, 长度7

| 89.47%| 100.00%  
      
    
    步骤=3, 评估ring分子, 长度7

| 85.00%| 87.50%  
      
    
    步骤=3, 评估branch分子

| 90.00%| 86.67%  
  
增加的branch增强数据, 提高了branch分子的识别率,

  

评估标红的错误分布: 

```
19   /opt/huangyan/mol-ai/images/branch_selfies/length_7/93.png  {
"selfies": "[C][Branch1][Ring2][C][C][O][O]"}          C(CCO)O      {
"selfies": "[O][Branch1][C][O][C][C][C]"}   O(O)CCC    0.117647   
mismatch
起点3, branch长度错误
[O][Branch1][C][O][C][C][C]
[O][O][C][C][C]
[C][Branch1][Ring1][C][C][O][O]
[C][Branch1][C][C][C][O][O]
[C][C][C][O][O]

5    /opt/huangyan/mol-ai/images/branch_selfies/length_7/87.png  {
"selfies": "[N][Branch1][Ring1][O][N][C][N]"}          N(ON)CN  {
"selfies": "[O][Branch1][Ring1][N][N][N][C]"}   O(NN)NC    0.142857   
mismatch
起点4, branch长度错误, 4-6轮换
[O][Branch1][Ring1][N][C][N][N]
[N][Branch1][C][N][O][N][C]
[N][N][O][N][C]
[N][Branch1][C][C][O][N][N]
[C][N][O][N][N]

25  /opt/huangyan/mol-ai/images/branch_selfies/length_7/144.png  {
"selfies": "[C][N][Branch1][Ring1][N][C][O]"}          CN(NC)O  {
"selfies": "[C][N][Branch1][Ring1][C][O][N]"}   CN(CO)N    0.235294   
mismatch
5-6轮换
[C][N][Branch1][C][N][C][O]
[N][Branch1][C][C][Branch1][C][N][C][O]
[C][Branch1][C][O][N][Branch1][C][C][N]
[O][C][N][Branch1][C][C][N]
[N][N][Branch1][C][C][C][O]
``` 

  

```
4   /opt/huangyan/mol-ai/images/branch_selfies/length_7/136.png      {
"selfies": "[C][Branch1][C][C][N][C][C]"}          C(C)NCC      {
"selfies": "[N][Branch1][C][C][C][C][C]"}   N(C)CCC    0.266667   
mismatch
起点4, 5-6轮换
[N][Branch1][C][C][C][C][C]
[C][N][C][C][C]
[C][Branch1][Ring1][C][C][N][C]
[C][Branch1][C][C][C][N][C]
[C][C][C][N][C]

14   /opt/huangyan/mol-ai/images/branch_selfies/length_7/25.png      {
"selfies": "[N][C][Branch1][C][O][C][N]"}          NC(O)CN      {
"selfies": "[O][C][C][Branch1][C][N][N]"}   OCC(N)N    0.357143   
mismatch
起点4(或镜像), 5-7镜像(分支内外镜像)
[O][C][C][Branch1][C][N][N]
[C][Branch1][C][O][C][Branch1][C][N][N]
[C][Branch1][C][N][Branch1][C][N][C][O]
[N][C][Branch1][C][N][C][O]
[N][C][Branch1][C][N][C][O]

1   /opt/huangyan/mol-ai/images/branch_selfies/length_7/125.png      {
"selfies": "[N][C][Branch1][C][C][C][N]"}          NC(C)CN  {
"selfies": "[N][C][Branch1][Ring1][C][C][N]"}   NC(CC)N    0.357143   
mismatch
branch长度错误
[N][C][Branch1][C][N][C][C]
[C][Branch1][C][N][Branch1][C][N][C][C]
[C][Branch1][C][C][C][Branch1][C][N][N]
[C][C][C][Branch1][C][N][N]
[N][C][Branch1][C][N][C][C]

16   /opt/huangyan/mol-ai/images/branch_selfies/length_7/22.png      {
"selfies": "[N][C][Branch1][C][O][O][N]"}          NC(O)ON      {
"selfies": "[N][C][Branch1][C][N][O][O]"}   NC(N)OO    0.357143   
mismatch
5-7轮换(分支内外轮换)
[N][C][Branch1][C][N][O][O]
[C][Branch1][C][N][Branch1][C][N][O][O]
[N][C][Branch1][C][N][O][O]
[O][Branch1][C][O][C][Branch1][C][N][N]
[O][O][C][Branch1][C][N][N]
``` 

  

额外: 在多步骤间, 也降低LR试试: 步骤1 (4e-4), 步骤2 (2e-4), 步骤3 (1e-4)

|  | 掺入2倍的branch增强数据(只有90个) |
| --- | --- |
      
    
    步骤=1, 评估普通分子, 长度7

| 84.21%  
      
    
    步骤=1, 评估ring分子, 长度7

| 82.50%  
      
    
    步骤=1, 评估branch分子

| 93.33%  
      
    
    步骤=2, 评估普通分子, 长度7

| 100.00%  
      
    
    步骤=2, 评估ring分子, 长度7

| 85.00%  
      
    
    步骤=2, 评估branch分子

| 83.33%  
      
    
    步骤=3, 评估普通分子, 长度7

| 94.74%  
      
    
    步骤=3, 评估ring分子, 长度7

| 87.50%  
      
    
    步骤=3, 评估branch分子

| 83.33%  
  
  

branch分子识别率SOTA, 模型的记忆能力是下降的. 先进行数据增强, 之后再穷举合适的LR.

  

数据增强: 生成不同起点的branch数据.

# 训练方式30

训练branch, 掺入 不同起点的branch数据, 主要步骤:

  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(70个), 掺入等量的部分普通分子, 掺入等量的ring分子, 掺入2倍的branch分子的rotate增强, LR=4e-4, 掺入等量的branch增强数据(添加[branch]操作符), 掺入等量的不同起点的branch数据
  - branch_selfies: 长度5(35个, 实际只有16个), 长度6(35个), 长度7(70个), 掺入等量的部分普通分子, 掺入等量的ring分子, 掺入2倍的branch分子的rotate增强, LR=2e-4, 掺入等量的branch增强数据(添加[branch]操作符), 掺入等量的不同起点的branch数据

commit: 4738f94438b3220259459096861c4679ff4a288a

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_4.train.json:8
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/raw/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/random_selfies_only_basic_atom/enhanced_by_image/length_3.train.json:4
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_7.train.json:140
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_7.train.json:140
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_6.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_6.train.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/raw/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/rotate_selfies_substring_from_branch_selfies/enhanced_by_image/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/raw/length_7.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/enhanced_by_image/length_7.json:70
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/raw/length_6.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/enhanced_by_image/length_6.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/raw/length_5.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/add_branch_op_from_random_selfies_only_basic_atom/enhanced_by_image/length_5.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_7.train.json:59
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_7.train.json:59
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_6.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_5.train.json:17
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_5.train.json:17

``` 

  

|  | 掺入等量的不同起点的branch数据 |
| --- | --- |
      
    
    步骤=1, 评估普通分子, 长度7

| 94.74%  
      
    
    步骤=1, 评估ring分子, 长度7

| 87.50%  
      
    
    步骤=1, 评估branch分子

| 90.00%  
      
    
    步骤=2, 评估普通分子, 长度7

| 84.21%  
      
    
    步骤=2, 评估ring分子, 长度7

| 92.50%  
      
    
    步骤=2, 评估branch分子

| 93.33%  
      
    
    步骤=3, 评估普通分子, 长度7

| 78.95%  
      
    
    步骤=3, 评估ring分子, 长度7

| 90.00%  
      
    
    步骤=3, 评估branch分子

| 93.33%  
  
  

看起来branch的准确率可以稳定在90%, 但仍然无法保持记忆准确率也在90%

  

对于标红部分进行错误分析: 

```
 10   /opt/huangyan/mol-ai/images/branch_selfies/length_7/36.png  {
"selfies": "[O][Branch1][Ring2][O][C][N][N]"}          O(OCN)N  {
"selfies": "[N][Branch1][Ring2][O][O][C][N]"}   N(OOC)N    0.095238   
mismatch
区别: 是 [C][O][O][N][N] 和 [N][C][O][O][N] 的branch形式
[N][Branch1][C][N][O][O][C]
[O][Branch1][Ring1][N][N][O][C]
[O][Branch1][C][C][O][N][N]
[C][O][O][N][N]
[N][N][O][O][C]

15  /opt/huangyan/mol-ai/images/branch_selfies/length_7/129.png  {
"selfies": "[O][Branch1][Ring2][O][C][N][N]"}          O(OCN)N      {
"selfies": "[O][Branch1][C][C][O][N][N]"}   O(C)ONN    0.095238   
mismatch
区别: 是 [C][O][O][N][N] 和 [N][C][O][O][N] 的branch形式
[O][Branch1][C][C][O][N][N]
[C][O][O][N][N]
[O][Branch1][Ring1][N][N][O][C]
[N][Branch1][C][N][O][O][C]
[N][N][O][O][C]

1   /opt/huangyan/mol-ai/images/branch_selfies/length_7/125.png  {
"selfies": "[N][C][Branch1][Ring1][C][N][C]"}          NC(CN)C  {
"selfies": "[N][C][Branch1][Ring1][C][C][N]"}   NC(CC)N    0.357143   
mismatch
6-7轮换(环内外轮换)
[N][C][Branch1][C][N][C][C]
[C][Branch1][C][N][Branch1][C][N][C][C]
[C][Branch1][C][C][C][Branch1][C][N][N]
[C][C][C][Branch1][C][N][N]
[N][C][Branch1][C][N][C][C]

``` 

  

```
 25  /opt/huangyan/mol-ai/images/branch_selfies/length_7/144.png      {
"selfies": "[N][Branch1][C][C][C][O][N]"}          N(C)CON  {
"selfies": "[C][N][Branch1][Ring1][C][O][N]"}   CN(CO)N    0.142857   
mismatch
镜像, 1-7轮换
[C][N][Branch1][C][N][C][O]
[N][Branch1][C][C][Branch1][C][N][C][O]
[C][Branch1][C][O][N][Branch1][C][C][N]
[O][C][N][Branch1][C][C][N]
[N][N][Branch1][C][C][C][O]

28   /opt/huangyan/mol-ai/images/branch_selfies/length_7/82.png  {
"selfies": "[O][Branch1][Ring1][O][N][C][O]"}          O(ON)CO  {
"selfies": "[O][Branch1][Ring1][O][O][N][C]"}   O(OO)NC    0.142857   
mismatch
5-6 轮换
[O][Branch1][Ring1][N][C][O][O]
[O][Branch1][C][O][O][N][C]
[O][O][O][N][C]
[N][Branch1][C][C][O][O][O]
[C][N][O][O][O]
``` 

  

# 整理

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
  2. 训练ring_selfies (训练方式21)
  3. 训练branch_selfies (训练方式30)

下一步: 在其上 提高长度8/9 的准确率
