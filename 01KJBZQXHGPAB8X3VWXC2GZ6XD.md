---
title: 20250127 - 使用ms-swift对Qwen2-VL进行微调 [20] - 调优长度8的效果[1]
confluence_page_id: 3344015
created_at: 2025-01-27T14:30:06+00:00
updated_at: 2025-01-31T07:42:55+00:00
---

# 基线

使用 [20250122 - 使用ms-swift对Qwen2-VL进行微调 [19] - 调优长度7的效果[3]] 最后一次的训练效果

使用随机生成的长度8的数据进行评估, commit: f37474d6d5eff9a076bbdbc8cc8408624cb6284c

评估结果: 

```
# epoch=1

评估长度7:
Correct: 84.00%, error_type
NO            77.0
mismatch      20.0
ring atoms     3.0

评估长度8:
Correct: 55.00%, error_type
NO            52.0
mismatch      47.0
ring atoms     1.0

# epoch=2

评估长度7:
Correct: 89.00%, error_type
NO            81.0
mismatch      16.0
ring atoms     3.0

评估长度8:
Correct: 68.00%, error_type
NO               58.0
mismatch         37.0
ring atoms        2.0
branch length     2.0
                  1.0

# epoch=3

评估长度7:
Correct: 90.00%, error_type
NO            83.0
mismatch      15.0
ring atoms     2.0

评估长度8:
Correct: 64.00%, error_type
NO            58.0
mismatch      40.0
ring atoms     2.0

# epoch=4

评估长度7:
Correct: 90.00%, error_type
NO            84.0
mismatch      12.0
ring atoms     4.0

评估长度8:
Correct: 72.00%, error_type
NO              61.0
mismatch        36.0
ring atoms       2.0
branch atoms     1.0
 
``` 

看起来, 扩大epoch可能长度8的准确率还能上升

先评估长度8的SOTA (epoch=4): [7.analyzed.2226.log](/assets/01KJBZQXHGPAB8X3VWXC2GZ6XD/7.analyzed.2226.log): 其中的错误分布大致为: 

```
ring内的原子乱序

一个原子识别错误, 其他正确

原子交换
原子交换, 键错误
键错误
键错误
键错误

最相似的是起点1([N][=C][N][C][Ring1][Ring2][Ring1][Ring1]), ring长度识别错误, ring的键错误

最相似的是起点3([C][#C][N][C][=N][O]), 多了两个原子, 有一个原子错误
最相似的是起点3([O][N][=N][N][C][#C]), 多了两个原子
最相似的是镜像([C][=N][C][C][#C][O]), 多了两个原子
最相似的是镜像([N][#C][C][#C][N][=O]), 多了两个原子
最相似的是镜像([N][O][N][=C][N][=N]), 多两个原子
最相似的是镜像([O][C][O][C][O][N]), 多了两个原子
最相似的是镜像([O][=N][N][=C][C][C]), 多了两个原子

最相似的是起点4([C][=N][N][C][=Branch1][C][=C][O]), 前四个原子正确
最相似的是镜像([C][#C][C][N][Branch1][C][C][N]), 前四个原子正确
最相似的是镜像([O][=C][=N][N][N][C][#C][N]), 前四个原子正确, 后四个原子轮换
最相似的是镜像([O][=C][C][#C][N][=C]), 前四个原子正确
最相似的是镜像([O][C][C][O][N][N][=C][C]), 后六个原子轮换

最相似的是镜像([C][N][N][Branch1][C][N][N][=O]), 两个原子远距离交换
最相似的是镜像([N][#C][N][=N][C][#C][C][=N]), 原子交换
最相似的是镜像([O][=C][N][O][N][C][=C][=N]), 原子交换
最相似的是镜像([N][N][O][C][=C][=C][O][C]), 原子轮换
最相似的是镜像([O][O][O][O][N][O][N][N]), 原子轮换

最相似的是起点4([O][N][N][C][Branch1][C][C][C]), branch位置错误
最相似的是镜像([N][C][C][N][Branch1][C][C][C]), branch位置错误
最相似的是镜像([O][C][C][Branch1][C][C][O][O]), branch位置错误
``` 

首先需要解决的是受提示词影响, 变换后会 多几个原子 的问题. 这个问题在长度7中就有发现但比较轻微. 

另: 扩大epoch: 并不能增加长度8的准确率

```
# epoch=4

评估长度5:
Correct: 98.00%, error_type
NO              97.0
mismatch         2.0
branch atoms     1.0

评估长度6:
Correct: 95.00%, error_type
NO               86.0
mismatch         11.0
ring atoms        1.0
branch length     1.0
branch atoms      1.0

评估长度7:
Correct: 92.00%, error_type
NO            84.0
mismatch      13.0
ring atoms     3.0

评估长度8:
Correct: 66.00%, error_type
NO               56.0
mismatch         42.0
ring atoms        1.0
branch length     1.0

# epoch=5

评估长度5:
Correct: 98.00%, error_type
NO              97.0
mismatch         2.0
branch atoms     1.0

评估长度6:
Correct: 95.00%, error_type
NO               88.0
mismatch         10.0
branch length     1.0
branch atoms      1.0

评估长度7:
Correct: 89.00%, error_type
NO            84.0
mismatch      14.0
ring atoms     2.0

评估长度8:
Correct: 70.00%, error_type
NO               61.0
mismatch         37.0
ring atoms        1.0
branch length     1.0

# epoch=6

评估长度5:
Correct: 96.00%, error_type
NO              95.0
mismatch         4.0
branch atoms     1.0

评估长度6:
Correct: 95.00%, error_type
NO              88.0
mismatch        11.0
branch atoms     1.0

评估长度7:
Correct: 92.00%, error_type
NO               81.0
mismatch         13.0
ring atoms        5.0
branch length     1.0

评估长度8:
Correct: 75.00%, error_type
NO            64.0
mismatch      35.0
ring atoms     1.0 
``` 

# 解决问题: "受提示词影响, 变换后会 多几个原子"

修改提示词, 去掉原子数限制, 重新评估各长度分子的结果

commit: 43137aa59bf1f51ae6dba1633a995f149811d277

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:120
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:120
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_6.train.json:57
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_6.train.json:57
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_7.train.json:59
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_7.train.json:59
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/raw/length_5.train.json:18
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_5.train.json:18
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/raw/length_7.train.json:258
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_7.train.json:258
``` 

训练效果: 

```
 # epoch=1

评估长度5:
Correct: 94.00%, error_type
NO               86.0
mismatch         11.0
branch length     2.0
ring atoms        1.0

评估长度6:
Correct: 95.00%, error_type
NO               79.0
mismatch          9.0
branch length     9.0
ring atoms        2.0
branch atoms      1.0

评估长度7:
Correct: 88.00%, error_type
NO               75.0
mismatch         17.0
branch length     6.0
ring atoms        2.0

评估长度8:
Correct: 72.00%, error_type
NO               51.0
mismatch         31.0
branch length    16.0
ring atoms        2.0

# epoch=2

评估长度5:
Correct: 100.00%, error_type
NO               93.0
branch length     3.0
mismatch          3.0
ring atoms        1.0

评估长度6:
Correct: 99.00%, error_type
NO               82.0
branch length    11.0
mismatch          5.0
ring atoms        1.0
branch atoms      1.0

评估长度7:
Correct: 89.00%, error_type
NO               77.0
mismatch         13.0
branch length     6.0
ring atoms        4.0

评估长度8:
Correct: 79.00%, error_type
NO               57.0
mismatch         21.0
branch length    17.0
ring atoms        5.0

# epoch=3

评估长度5:
Correct: 99.00%, error_type
NO               90.0
branch length     4.0
mismatch          4.0
ring atoms        2.0

评估长度6:
Correct: 98.00%, error_type
NO               83.0
branch length     9.0
mismatch          7.0
branch atoms      1.0

评估长度7:
Correct: 92.00%, error_type
NO               79.0
mismatch         11.0
branch length     7.0
ring atoms        3.0

评估长度8:
Correct: 81.00%, error_type
NO               63.0
mismatch         19.0
branch length    16.0
ring atoms        2.0

# epoch=4

评估长度5:
Correct: 98.00%, error_type
NO               90.0
mismatch          5.0
branch length     4.0
ring atoms        1.0

评估长度6:
Correct: 99.00%, error_type
NO               82.0
branch length     8.0
mismatch          7.0
ring atoms        2.0
branch atoms      1.0

评估长度7:
Correct: 94.00%, error_type
NO               83.0
mismatch          7.0
branch length     7.0
ring atoms        3.0

评估长度8:
Correct: 75.00%, error_type
NO               51.0
mismatch         26.0
branch length    17.0
ring atoms        6.0
``` 

对于长度5/6/7/8, 一些偏差移动到了branch length (但不完全是错误, 需要修正评估脚本, 只对错误case进行分析)

对长度8的准确率, 提高到了81% (epoch=3), 分析错误case: [12.analyzed.1753.log](/assets/01KJBZQXHGPAB8X3VWXC2GZ6XD/12.analyzed.1753.log)

```
最相似的是镜像([N][C][C][C][N][Ring1][Ring2][O]), 环内原子乱序
ring环原子乱序, 键错误
环内原子乱序
环内原子乱序, 键错误
环内原子键错误

(ring+环)整体平移
最相似的是镜像([C][C][#C][C][C][N][Ring1][Ring1]), (ring+环)整体平移

最相似的是镜像([C][=C][C][C][C][#C][N][=N]), 两次原子交换
最相似的是镜像([N][=C][C][C][C][N]), 原子交换
最相似的是镜像([C][C][=C][=N][N][=C][N][=O]), 原子交换
原子交换

最相似的是镜像([C][C][C][C][N][N][=N][C]), 三原子平移

 
键错误, 三原子轮换
键错误
键错误

多一个[C]

环识别错误, 远距离原子交换, 键错误

涉及ring嵌套, 最相似的是起点1 ([N][=C][N][C][Ring1][Ring2][Ring1][Ring1]), 缺少最后两个ring操作符

最相似的是起点4([O][NH1][N][Ring1][Ring1][O][N][N]), 缺一个[O]
``` 

分析当前的数据分布: 

  - chain_selfies: 链式生成的分子
    - 增强: 镜像
    - 增强: 交换原子
  - branch_selfies: 分支构成的分子 (包括了 在 前主链/分支/后主链 中的某一个进行随机原子变化, 但不包括键的变化)
    - 增强: 随机起点
  - ring_selfies: 环构成的分子 (包括了 在 前主链/环/后主链 中的某一个进行随机原子变化, 但不包括键的变化)
    - 增强: 镜像
    - 增强: 将分子和镜像分子, 进行随机的键 重配

# 增加ring环内的分子随机化

commit: 19956b10a62b0d2cd9abe55f49d60633271c1380

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:120
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:120
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_6.train.json:57
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_6.train.json:57
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_7.train.json:59
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_7.train.json:59
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/raw/length_5.train.json:18
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_5.train.json:18
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/raw/length_7.train.json:258
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_7.train.json:258
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/raw/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/enhanced_by_image/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/raw/length_7.train.json:276
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/enhanced_by_image/length_7.train.json:276
``` 

训练效果: 效果并没有提升

```
 # epoch=1

评估长度5:
Correct: 94.00%, error_type
NO               88.0
mismatch          6.0
branch length     4.0
ring atoms        2.0

评估长度6:
Correct: 94.00%, error_type
NO               87.0
mismatch          6.0
branch length     4.0
ring atoms        3.0

评估长度7:
Correct: 86.00%, error_type
NO               73.0
mismatch         15.0
branch length     9.0
ring atoms        2.0
branch atoms      1.0

评估长度8:
Correct: 71.00%, error_type
NO               57.0
mismatch         31.0
branch length    10.0
ring length       1.0
ring atoms        1.0

# epoch=2

评估长度5:
Correct: 97.00%, error_type
NO               87.0
branch length     7.0
mismatch          5.0
ring atoms        1.0

评估长度6:
Correct: 98.00%, error_type
NO               90.0
branch length     4.0
mismatch          3.0
ring atoms        3.0

评估长度7:
Correct: 90.00%, error_type
NO               76.0
mismatch         11.0
branch length     9.0
ring atoms        3.0
branch atoms      1.0

评估长度8:
Correct: 77.00%, error_type
NO               63.0
mismatch         24.0
branch length    11.0
ring atoms        1.0
ring length       1.0

# epoch=3

评估长度5:
Correct: 93.00%, error_type
NO               83.0
mismatch          9.0
branch length     8.0

评估长度6:
Correct: 97.00%, error_type
NO               90.0
mismatch          6.0
branch length     4.0

评估长度7:
Correct: 90.00%, error_type
NO               78.0
mismatch         11.0
branch length     8.0
ring atoms        2.0
branch atoms      1.0

评估长度8:
Correct: 75.00%, error_type
NO               61.0
mismatch         25.0
branch length    12.0
ring length       1.0
ring atoms        1.0

# epoch=4

评估长度5:
Correct: 96.00%, error_type
NO               81.0
branch length    10.0
mismatch          8.0
ring atoms        1.0

评估长度6:
Correct: 99.00%, error_type
NO               89.0
branch length     5.0
mismatch          4.0
ring atoms        2.0

评估长度7:
Correct: 93.00%, error_type
NO               80.0
branch length     9.0
mismatch          8.0
branch atoms      2.0
ring atoms        1.0

评估长度8:
Correct: 69.00%, error_type
NO               54.0
mismatch         31.0
branch length    12.0
ring atoms        2.0
ring length       1.0
``` 

分析SOTA的错误分布: [12.analyzed.1129.log](/assets/01KJBZQXHGPAB8X3VWXC2GZ6XD/12.analyzed.1129.log)

```
最相似的是镜像([C][=C][=C][Ring1][Ring1][N][C][=N]), 键错误
最相似的是镜像([N][O][O][N][C][N][C][=O]), 键错误
最相似的是镜像([C][O][N][C][=C][C][C][#N]), 键错误
最相似的是镜像([N][#C][N][=C][C][C][C][=N]), 键错误
键错误
键错误
最相似的是镜像([N][=C][=C][C][=C][C][Ring1][Ring1]), 环内键错误
环内键错误
环内键错误
环内键错误

ring位置错误

ring位置错误, 缺少[N]
键错误, 少一个[N]
最相似的是镜像([N][C][C][C][N][Ring1][Ring2][O]), 环范围少一个[C]
最相似的是镜像([N][#C][N][=N][C][#C][C][=N]), 少一个[C]
最相似的是镜像([N][#C][C][=N][N][C][=C][=N]), 少一个[C]

最相似的是镜像([C][=C][C][C][N][C][Ring1][Branch1]), 原子交换, 缺少ring
最相似的是镜像([C][C][=C][C][Ring1][Ring2][=C][O]), 缺少ring    

缺少branch结构, 分支内键错误
缺少branch结构, 分支内键错误
最相似的是起点4([C][N][=C][C][=Branch1][C][=N][N]), 缺少branch结构
最相似的是起点4([N][C][Branch1][C][O][N][C][=O]), 缺少branch结构, 分支位置错误
最相似的是起点2([N][=Branch1][=Branch1][=C][C][=C][O][N][C][N]), 缺少branch结构, 分支内原子交换, 键错误

原子交换, 键错误
最相似的是镜像([C][=C][N][N][=C][Ring1][Ring1][C]), 环内原子交换

``` 

环内原子乱序的现象消失, 主要错误变成键错误. 另外: 原子缺失的现象在变多, 缺少branch的现象也在变多.

另: 当前的增强数据分布: 

  - chain_selfies: 链式生成的分子
    - 增强: 镜像
    - 增强: 交换原子
  - branch_selfies: 分支构成的分子 (包括了 在 前主链/分支/后主链 中的某一个进行随机原子变化, 但不包括键的变化)
    - 增强: 随机起点
  - ring_selfies: 环构成的分子 (包括了 在 前主链/环/后主链 中的某一个进行随机原子变化, 但不包括键的变化)
    - 增强: 镜像
    - 增强: 将分子和镜像分子, 进行随机的键 重配
    - 增强: 将分子和镜像分子, 进行原子 随机化

# 将chain_selfies, 进行随机的键 重配

commit: faf58466e67f05364f356e8175a2f96e385cc633

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/raw/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/raw/length_7.train.json:120
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/branch_selfies/enhanced_by_image/length_7.train.json:120
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_6.train.json:57
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_6.train.json:57
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/raw/length_7.train.json:59
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_start_point_from_branch_selfies/enhanced_by_image/length_7.train.json:59
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/raw/length_5.train.json:18
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_5.train.json:18
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/raw/length_7.train.json:258
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_7.train.json:258
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_chain_selfies_and_mirror/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_chain_selfies_and_mirror/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_chain_selfies_and_mirror/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_chain_selfies_and_mirror/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_chain_selfies_and_mirror/raw/length_7.train.json:694
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_bond_from_chain_selfies_and_mirror/enhanced_by_image/length_7.train.json:694
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/raw/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/enhanced_by_image/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/raw/length_7.train.json:276
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.prompt_without_length/randomize_ring_atoms_from_ring_selfies_and_mirror/enhanced_by_image/length_7.train.json:276
``` 

训练效果: 

```
# epoch=1

评估长度5:
Correct: 94.00%, error_type
NO               90.0
mismatch          8.0
branch length     2.0

评估长度6:
Correct: 98.00%, error_type
NO               87.0
branch length    10.0
mismatch          3.0

评估长度7:
Correct: 90.00%, error_type
NO               81.0
mismatch         12.0
ring atoms        3.0
branch length     3.0
ring length       1.0

评估长度8:
Correct: 71.00%, error_type
NO               61.0
mismatch         29.0
branch length    10.0

# epoch=2

评估长度5:
Correct: 99.00%, error_type
NO               95.0
mismatch          3.0
branch length     2.0

评估长度6:
Correct: 99.00%, error_type
NO               87.0
branch length     9.0
mismatch          3.0
branch atoms      1.0

评估长度7:
Correct: 93.00%, error_type
NO               84.0
mismatch          9.0
branch length     3.0
ring atoms        2.0
ring length       1.0
branch atoms      1.0

评估长度8:
Correct: 73.00%, error_type
NO               62.0
mismatch         28.0
branch length    10.0

 
# epoch=3

评估长度5:
Correct: 99.00%, error_type
NO               92.0
branch length     5.0
mismatch          3.0

评估长度6:
Correct: 99.00%, error_type
NO               86.0
branch length    10.0
mismatch          3.0
ring atoms        1.0

评估长度7:
Correct: 93.00%, error_type
NO               84.0
mismatch         10.0
branch length     3.0
ring length       1.0
ring atoms        1.0
branch atoms      1.0

评估长度8:
Correct: 60.00%, error_type
NO               50.0
mismatch         41.0
branch length     9.0
``` 

分析epoch=2的长度8的分析错误: [8.analyzed.2229.log](/assets/01KJBZQXHGPAB8X3VWXC2GZ6XD/8.analyzed.2229.log)

```
最相似的是镜像([O][O][C][N][=N][N][N][=C]), 键错误
键错误
键错误
键错误
键错误
键错误
环内键错误
环内键错误

最相似的是起点4([O][C][C][Branch1][C][O][=N][O]), 缺少branch结构, 原子交换
最相似的是长度4([N][=N][N][Branch1][C][O][C][=N]), 缺少branch结构, 将分支内的原子放在了最后
最相似的是镜像([C][=C][C][Branch1][C][O][C][C]), 缺少branch结构, 将分支内的原子放在了最后
缺少branch结构, 原子交换
缺少branch结构

缺少ring结构

最相似的是镜像([N][=N][C][=N][N][C][Ring1][Ring2]), ring长度错误, 环内原子乱序
最相似的是镜像([O][=N][C][N][O][N][Ring1][Ring2]), 环长度错误, 环内原子交换
最相似的是镜像([C][C][=C][C][C][O][Ring1][Ring2]), 环内原子乱序

最相似的是镜像([O][=N][N][C][C][O][N][C]), [N]原子位置错误
[C]原子位置错误, 键错误
[C]原子位置错误

最相似的是镜像([O][O][O][N][=C][N][=C][=C]), 缺少[O]
最相似的是镜像([O][O][O][C][Ring1][Ring2][N][=C]), 缺少[O]
最相似的是镜像([N][N][=N][N][C][C][=N][N]), 缺少[N]
最相似的是镜像([N][C][#C][N][N][C][C][=N]), 缺少[C]
缺少[C]
缺少[C]
缺少[O]
``` 

(增强方式)随机的键重配 并不能修正键错误?

# 一些想法

  - 需要减少训练成本:
    - 生成一个基模型, 识别[C][O][N]和双键/三键, 在此模型上, 再进行Ring和Branch的训练. (思考: 排除Ring和Branch, 简单的结构识别起来难度较低)
    - 目前的模式是: step 1(epoch=2), step 2(epoch=2), .... 尝试调整成 step 1(epoch=1), step 2(epoch=1), ..., step 1(epoch=1), step 2(epoch=1), .... 这样可以逐步迭代, 得出哪个epoch效果比较好

# 临时增加一个测试: 尝试epoch逐步迭代

使用上面最后一次训练的数据, 改成epoch逐步迭代: 

```
# epoch=1

评估长度5:
Correct: 95.00%, error_type
NO               89.0
mismatch          7.0
branch length     4.0

评估长度6:
Correct: 91.00%, error_type
NO               81.0
mismatch         10.0
branch length     9.0

评估长度7:
Correct: 94.00%, error_type
NO               85.0
mismatch          6.0
branch length     4.0
ring atoms        3.0
ring length       1.0
branch atoms      1.0

评估长度8:
Correct: 74.00%, error_type
NO               63.0
mismatch         26.0
branch length    10.0
ring atoms        1.0

# epoch+1=2

评估长度5:
Correct: 98.00%, error_type
NO               92.0
branch length     4.0
mismatch          4.0

评估长度6:
Correct: 98.00%, error_type
NO               83.0
branch length    11.0
mismatch          4.0
ring atoms        2.0

评估长度7:
Correct: 94.00%, error_type
NO               84.0
mismatch          7.0
branch length     4.0
ring atoms        3.0
ring length       1.0
branch atoms      1.0

评估长度8:
Correct: 80.00%, error_type
NO               68.0
mismatch         20.0
branch length    10.0
ring atoms        2.0

# epoch+1=3

评估长度5:
Correct: 96.00%, error_type
NO               91.0
mismatch          5.0
branch length     4.0

评估长度6:
Correct: 100.00%, error_type
NO               86.0
branch length    10.0
ring atoms        2.0
branch atoms      1.0
mismatch          1.0

评估长度7:
Correct: 94.00%, error_type
NO               84.0
mismatch          8.0
branch length     4.0
ring atoms        2.0
ring length       1.0
branch atoms      1.0

评估长度8:
Correct: 78.00%, error_type
NO               67.0
mismatch         22.0
branch length    11.0
``` 

训练效果 比之前好, 多种epoch尝试也会大幅节省时间, 之后改成这种方式运作

评估SOTA (epoch=2)的错误分布: [8.analyzed.1423.log](/assets/01KJBZQXHGPAB8X3VWXC2GZ6XD/8.analyzed.1423.log)

```
最相似的是镜像([C][#C][N][=C][=C][Ring1][Branch1][N]), 键错误
最相似的是镜像([N][=N][C][=N][N][C][Ring1][Ring2]), 键错误
最相似的是镜像([N][N][C][=N][N][N][C][=C]), 键错误
键错误
键错误

缺少branch结构
缺少branch结构
最相似的是起点4([O][=N][N][C][Branch1][C][C][=O]), 缺少branch结构, branch结构被放到了最后
最相似的是起点4([N][N][Branch1][C][N][C][O][O]), 缺少branch结构
最相似的是起点4([O][C][C][Branch1][C][O][=N][O]), 缺少branch结构, branch结构被放到了最后

最相似的镜像([C][#C][N][Branch1][C][C][N][=O]), 分支内分子错误

最相似的是镜像([C][C][=C][C][C][O][Ring1][Ring2]), 环内原子交换
最相似的是镜像([N][C][=C][C][C][N][Ring1][Branch1]), 环内原子乱序
环内原子交换, 整体分子轮换

整个分子轮换

[O]位置错误, 与连续[O]有关
最相似的是镜像([C][#C][C][C][N][N][Ring1][Ring2]), [C]原子位置错误, 与整个环交换; 与连续[C]有关

少一个[O]
少一个[C], 与连续[C]有关
最相似的是镜像([O][O][O][C][Ring1][Ring2][N][=C]), 少一个[O], 与连续[O]有关
``` 

从错误分布来看, "将chain_selfies, 进行随机的键 重配"的数据增强好像并没有显著作用
