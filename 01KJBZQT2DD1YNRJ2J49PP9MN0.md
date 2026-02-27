---
title: 20250122 - 使用ms-swift对Qwen2-VL进行微调 [19] - 调优长度7的效果[3]
confluence_page_id: 3343957
created_at: 2025-01-22T15:45:02+00:00
updated_at: 2025-01-27T13:52:42+00:00
---

# 增加[ring]的规范数据

commit: 66107a8679fe50aaa3604ae6794ccfbd1aeea43c

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_6.train.json:340
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_6.train.json:340
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_6.train.json:340
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_6.train.json:340
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:27
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:27
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:93
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:93
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:316
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:316
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_6.train.json:40
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_6.train.json:40

``` 

注意: 在"cot_with_CNO_count.enhance_by_randomize_selfies" 的增强方法中, 其中的ring_selfies数据并没有换成新的, 需要修正.

以下结果是未修正数据时的结果: 评估结果: 

```
# epoch=1

评估长度5:
Correct: 88.00%, Predict selfies valid: 97.00%, error_type
NO                  87.0
error_atom_seq       7.0
error_ring           3.0
error_bond_count     2.0
error_branch         1.0

评估长度6:
Correct: 82.00%, Predict selfies valid: 99.00%, error_type
NO                  80.0
error_atom_seq       8.0
error_bond_count     4.0
error_ring           3.0
error_atom_count     2.0
error_bond_seq       2.0
error_branch         1.0

评估长度7: 
Correct: 62.00%, Predict selfies valid: 91.00%, error_type
NO                  60.0
error_atom_seq      12.0
error_branch         9.0
error_ring           8.0
error_atom_count     5.0
error_bond_seq       4.0
error_other          1.0
error_bond_count     1.0

# epoch=2

评估长度5:
Correct: 94.00%, Predict selfies valid: 97.00%, error_type
NO                  92.0
error_atom_seq       4.0
error_bond_count     2.0
error_branch         1.0
error_ring           1.0

评估长度6:
Correct: 89.00%, Predict selfies valid: 97.00%, error_type
NO                  86.0
error_atom_seq       6.0
error_ring           4.0
error_bond_count     3.0
error_atom_count     1.0

评估长度7: 
Correct: 75.00%, Predict selfies valid: 95.00%, error_type
NO                  72.0
error_atom_seq      13.0
error_branch         6.0
error_ring           5.0
error_bond_count     2.0
error_atom_count     1.0
error_bond_seq       1.0

# epoch=3

评估长度5:
Correct: 95.00%, Predict selfies valid: 97.00%, error_type
NO                  90.0
error_atom_seq       6.0
error_branch         1.0
error_atom_count     1.0
error_ring           1.0
error_bond_count     1.0

评估长度6:
Correct: 92.00%, Predict selfies valid: 97.00%, error_type
NO                  87.0
error_atom_seq       6.0
error_ring           4.0
error_bond_seq       1.0
error_branch         1.0
error_bond_count     1.0

评估长度7: 
Correct: 78.00%, Predict selfies valid: 94.00%, error_type
NO                  74.0
error_atom_seq      10.0
error_branch         8.0
error_ring           5.0
error_atom_count     1.0
error_bond_count     1.0
error_bond_seq       1.0

``` 

规范[Ring]数据是有效的, SOTA上升至78%

分析SOTA的评估日志: [9.1508.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/9.1508.log)

其中与Ring相关的错误:

  - 3个, ring长度识别错误 (涉及[Ring1][Branch1], 在长度5/6的训练数据中不存在)
  - 2个, ring键错误(涉及[=Ring1], 在训练数据中是randomize中有, 需要修正成规范数据)
  - 4个, ring位置错误
  - 1个, ring长度识别错误

下一步:

  - 前两种错误认为在扩大 分子长度 的训练后, 会变好, 现在不处理
  - ring位置错误, 尝试通过引入镜像增强来尝试修正
  - 对randomize增强, 将数据修正成规范数据

# 引入ring分子的镜像增强

commit: 05f0ed8f6d1934c6c89bc662773cf811b10e84fa

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_6.train.json:340
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_6.train.json:340
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_6.train.json:340
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_6.train.json:340
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:27
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:27
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:93
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:93
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:316
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:316
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_6.train.json:80 
``` 

```
# epoch=1
 
评估长度5:
Correct: 92.00%, Predict selfies valid: 96.00%, error_type
NO                  91.0
error_branch         3.0
error_atom_seq       2.0
error_bond_count     2.0
error_ring           1.0
error_atom_count     1.0

评估长度6:
Correct: 83.00%, Predict selfies valid: 96.00%, error_type
NO                  80.0
error_atom_seq       8.0
error_bond_count     4.0
error_atom_count     3.0
error_ring           3.0
error_bond_seq       1.0
error_branch         1.0

评估长度7: 
Correct: 64.00%, Predict selfies valid: 92.00%, error_type
NO                  63.0
error_atom_seq      15.0
error_branch         8.0
error_ring           5.0
error_atom_count     4.0
error_bond_count     3.0
                     1.0
error_bond_seq       1.0
 
 
# epoch=2

评估长度5:
Correct: 93.00%, Predict selfies valid: 98.00%, error_type
NO                  92.0
error_branch         3.0
error_atom_seq       3.0
error_bond_count     2.0

评估长度6:
Correct: 87.00%, Predict selfies valid: 99.00%, error_type
NO                  83.0
error_atom_seq      10.0
error_ring           4.0
error_bond_seq       1.0
error_branch         1.0
error_atom_count     1.0

评估长度7: 
Correct: 72.00%, Predict selfies valid: 94.00%, error_type
NO                  69.0
error_atom_seq      13.0
error_ring           7.0
error_branch         6.0
error_bond_count     3.0
error_atom_count     2.0
 
 
# epoch=3

评估长度5:
Correct: 92.00%, Predict selfies valid: 97.00%, error_type
NO                  90.0
error_atom_seq       3.0
error_branch         3.0
error_bond_count     3.0
error_ring           1.0

评估长度6:
Correct: 93.00%, Predict selfies valid: 98.00%, error_type
NO                  86.0
error_atom_seq       9.0
error_ring           2.0
error_atom_count     1.0
error_branch         1.0
error_bond_count     1.0

评估长度7: 
Correct: 69.00%, Predict selfies valid: 95.00%, error_type
NO                  66.0
error_atom_seq      17.0
error_ring           9.0
error_branch         4.0
error_atom_count     2.0
error_bond_seq       1.0
error_bond_count     1.0
 
# epoch=4

评估长度5:
Correct: 96.00%, Predict selfies valid: 99.00%, error_type
NO                  95.0
error_atom_seq       1.0
error_branch         1.0
                     1.0
error_ring           1.0
error_bond_count     1.0

评估长度6:
Correct: 88.00%, Predict selfies valid: 99.00%, error_type
NO                  87.0
error_atom_seq       7.0
error_ring           4.0
error_bond_count     2.0

评估长度7: 
Correct: 73.00%, Predict selfies valid: 94.00%, error_type
NO                  69.0
error_atom_seq      15.0
error_branch         7.0
error_ring           5.0
error_atom_count     2.0
error_bond_count     2.0

``` 

分析epoch=2时, 长度7的评估结果: [6.analyzed.2353.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/6.analyzed.2353.log) : ring相关的错误:

  - 2个: [ring1][ring1]起始位置错误, 将第四位识别为第五位: 训练数据中, [ring1][ring1]都是最后一位或者倒数第二位, 没有出现在第四位 (倒数第三位), 认为增加训练数据长度后, 可能会改善, 暂不用处理
  - 3个: [ring1][branch1]错误: 训练数据中没有[ring1][branch1], 认为增加训练数据长度后, 可能会改善, 暂不用处理
  - 3个: [ring1][ring2]错误, 被识别成了[ring1][ring1]
    - 其中有2个, [ring1][ring2] 出现在倒数第二位, 在训练数据中没有该位置, 认为增加训练数据长度后, 可能会改善, 暂不用处理
    - 其中有1个, 确实是识别错误
  - 1个: ring未识别

分析epoch=4时, 长度7的评估结果: [12.analyzed.0008.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/12.analyzed.0008.log): ring相关的错误:

  - 1个: [Ring1][Ring1]起始位置错误, 将第四位识别为第五位, 认为增加训练数据长度后, 可能会改善, 暂不用处理
  - 3个: [Ring1][Branch1]长度识别错误: 训练数据中没有[ring1][branch1], 认为增加训练数据长度后, 可能会改善, 暂不用处理
  - 2个: [Ring1][Ring2]起始位置错误: [ring1][ring2] 出现在倒数第二位, 在训练数据中没有该位置, 认为增加训练数据长度后, 可能会改善, 暂不用处理
  - 1个: ring未识别
  - 1个: [Ring1][Ring2]长度识别错误

从错误分布上, 看不出ring镜像增强带来的收益, 暂时将其去掉

得尝试增加训练长度, 来看ring的一些可修整的错误是否得以修正

# 增加长度7的数据作为训练数据

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_7.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_7.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_7.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_7.train.json:80

``` 

训练效果: 

  - 注意: 长度8的评估数据不是random, 而是用chain进行链式生成, 所以其值会虚高 (链中前半部分的数据已经在训练数据中). 长度8的评估数据不能作为准确值.

```
# epoch=1
 
评估长度5:
Correct: 93.00%, Predict selfies valid: 98.00%, error_type
NO                  91.0
error_branch         3.0
error_bond_count     3.0
error_atom_seq       2.0
error_ring           1.0

 
评估长度6:
Correct: 88.00%, Predict selfies valid: 98.00%, error_type
NO                  83.0
error_atom_seq      10.0
error_ring           4.0
error_bond_count     2.0
error_bond_seq       1.0

 
评估长度7:
Correct: 78.00%, Predict selfies valid: 90.00%, error_type
NO                  76.0
error_atom_seq      10.0
error_ring           9.0
error_branch         4.0
error_atom_count     1.0

 
评估长度8 (已知会虚高):
Correct: 89.80%, Predict selfies valid: 100.00%, error_type
NO                  89.795918
error_atom_seq       5.102041
error_bond_seq       3.061224
error_atom_count     1.020408
error_bond_count     1.020408

 # epoch=2

评估长度5:
Correct: 92.00%, Predict selfies valid: 100.00%, error_type
NO                  90.0
error_branch         4.0
error_atom_seq       3.0
error_bond_count     3.0

评估长度6:
Correct: 93.00%, Predict selfies valid: 99.00%, error_type
NO                  88.0
error_ring           5.0
error_atom_seq       3.0
error_bond_count     3.0
error_bond_seq       1.0

评估长度7:
Correct: 82.00%, Predict selfies valid: 91.00%, error_type
NO                  78.0
error_atom_seq      14.0
error_ring           4.0
error_atom_count     2.0
error_bond_count     1.0
error_branch         1.0

评估长度8 (已知会虚高):
Correct: 90.82%, Predict selfies valid: 97.96%, error_type
NO                  90.816327
error_atom_seq       5.102041
error_bond_seq       3.061224
error_bond_count     1.020408
 
# epoch=3

评估长度5:
Correct: 95.00%, Predict selfies valid: 99.00%, error_type
NO                  94.0
error_branch         3.0
error_atom_seq       2.0
error_bond_count     1.0

评估长度6:
Correct: 94.00%, Predict selfies valid: 98.00%, error_type
NO                  90.0
error_bond_count     4.0
error_atom_seq       3.0
error_branch         2.0
error_ring           1.0

评估长度7:
Correct: 78.00%, Predict selfies valid: 94.00%, error_type
NO                  78.0
error_atom_seq      12.0
error_ring           6.0
error_bond_count     2.0
error_branch         2.0

评估长度8 (已知会虚高):
Correct: 90.82%, Predict selfies valid: 98.98%, error_type
NO                  90.816327
error_atom_seq       3.061224
error_atom_count     3.061224
                     1.020408
error_bond_count     1.020408
error_bond_seq       1.020408

# epoch=4

评估长度5:
Correct: 94.00%, Predict selfies valid: 98.00%, error_type
NO                  92.0
error_atom_seq       3.0
error_ring           2.0
error_bond_count     2.0
error_branch         1.0

评估长度6:
Correct: 93.00%, Predict selfies valid: 97.00%, error_type
NO                  90.0
error_bond_count     4.0
error_atom_seq       3.0
error_branch         2.0
error_ring           1.0

评估长度7:
Correct: 84.00%, Predict selfies valid: 95.00%, error_type
NO                  77.0
error_atom_seq      11.0
error_ring           6.0
error_bond_count     4.0
error_atom_count     1.0
error_branch         1.0

评估长度8 (已知会虚高):
Correct: 92.86%, Predict selfies valid: 100.00%, error_type
NO                  92.857143
error_bond_seq       3.061224
error_atom_seq       2.040816
error_bond_count     1.020408
error_atom_count     1.020408

``` 

就算加入了 长度7的数据 作为训练数据 (相当于弱化推理能力, 而使用相似性匹配), 准确率也只能提升到84%. 

# 增加长度7的ring的数据量 (80 -> 160)

分析评估中的错误case, 大部分是ring相关的错误, **考虑 增加长度7的ring的数据量 (80 -> 160)**, 数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_7.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_7.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_7.train.json:160

``` 

训练效果: 

```
# epoch=1
评估长度5: 
Correct: 91.00%, Predict selfies valid: 97.00%, error_type
NO                  88.0
error_atom_seq       5.0
error_ring           3.0
error_bond_count     3.0
error_branch         1.0

评估长度6: 
Correct: 89.00%, Predict selfies valid: 100.00%, error_type
NO                  86.0
error_atom_seq       8.0
error_ring           4.0
error_bond_count     1.0
error_other          1.0

评估长度7:  
Correct: 79.00%, Predict selfies valid: 94.00%, error_type
NO                  73.0
error_atom_seq      14.0
error_ring           8.0
error_bond_count     2.0
error_branch         2.0
error_bond_seq       1.0

评估长度8:  
Correct: 83.67%, Predict selfies valid: 97.96%, error_type
NO                  83.673469
error_atom_seq       9.183673
error_bond_count     6.122449
error_bond_seq       1.020408

# epoch=2

评估长度5: 
Correct: 93.00%, Predict selfies valid: 98.00%, error_type
NO                  92.0
error_branch         3.0
error_bond_count     3.0
error_atom_seq       2.0

评估长度6: 
Correct: 88.00%, Predict selfies valid: 98.00%, error_type
NO                  84.0
error_atom_seq       7.0
error_bond_count     5.0
error_ring           4.0

评估长度7: 
Correct: 79.00%, Predict selfies valid: 96.00%, error_type
NO                  74.0
error_atom_seq      13.0
error_ring           9.0
error_bond_count     2.0
error_bond_seq       2.0

评估长度8: 
Correct: 94.90%, Predict selfies valid: 98.98%, error_type
NO                  93.877551
error_atom_seq       3.061224
error_bond_count     2.040816
error_atom_count     1.020408

# epoch=3

评估长度5: 
Correct: 94.00%, Predict selfies valid: 99.00%, error_type
NO                  92.0
error_branch         4.0
error_atom_seq       2.0
error_bond_count     2.0

评估长度6: 
Correct: 88.00%, Predict selfies valid: 98.00%, error_type
NO                  85.0
error_bond_count     5.0
error_atom_seq       4.0
error_ring           4.0
error_other          1.0
error_bond_seq       1.0

评估长度7: 
Correct: 83.00%, Predict selfies valid: 97.00%, error_type
NO                  79.0
error_atom_seq      10.0
error_ring           7.0
error_bond_count     2.0
error_branch         2.0
 
评估长度8: 
Correct: 91.84%, Predict selfies valid: 100.00%, error_type
NO                  91.836735
error_atom_seq       6.122449
error_bond_seq       1.020408
error_atom_count     1.020408

# epoch=4

评估长度5: 
Correct: 94.00%, Predict selfies valid: 100.00%, error_type
NO                  93.0
error_bond_count     3.0
error_atom_seq       2.0
error_branch         1.0
error_ring           1.0

评估长度6: 
Correct: 92.00%, Predict selfies valid: 98.00%, error_type
NO                  89.0
error_bond_count     4.0
error_ring           3.0
error_atom_seq       3.0
error_bond_seq       1.0

评估长度7: 
Correct: 84.00%, Predict selfies valid: 96.00%, error_type
NO                  81.0
error_atom_seq      10.0
error_ring           6.0
error_bond_count     2.0
                     1.0

评估长度8: 
Correct: 92.86%, Predict selfies valid: 100.00%, error_type
NO                92.857143
error_atom_seq     7.142857
``` 

增加长度7的ring的数据量, 无法提高评估的结果, 分析SOTA (epoch=4) 的错误case: [15.analyzed.2039.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/15.analyzed.2039.log), 其中: 

  - 5个: ring镜像后相似
  - 1个: ring位置正确, 其他原子顺序错误
  - 2个: ring相关, 键错误
  - 6个: branch错误

在分析(epoch=1)的错误case: [3.analyzed.2048.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/3.analyzed.2048.log), 其中:

  - 1个: ring长度错误
  - 1个: ring未识别
  - 3个: ring镜像后相似
  - 1个: ring平移
  - 2个: ring相关, 原子交换
  - 2个: ring相关, 键错误
  - 不统计branch错误

看起来, 增加ring镜像数据 (同样的图片, 识别出selfies表达式的镜像), 可能会有用. 再次尝试ring镜像数据.

# 增加ring镜像数据 (同样的图片, 识别出selfies表达式的镜像)

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_7.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_7.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_7.train.json:160
``` 

训练效果: 

```
# epoch=1
评估长度5: 
Correct: 89.00%, Predict selfies valid: 97.00%, error_type
NO                  88.0
error_atom_seq       4.0
error_ring           4.0
error_bond_count     3.0
error_other          1.0

评估长度6: 
Correct: 90.00%, Predict selfies valid: 97.00%, error_type
NO                  83.0
error_atom_seq      10.0
error_bond_count     3.0
error_branch         2.0
error_ring           2.0

评估长度7: 
Correct: 77.00%, Predict selfies valid: 94.00%, error_type
NO                  75.0
error_atom_seq      13.0
error_ring           6.0
error_branch         4.0
error_bond_count     1.0
error_atom_count     1.0

评估长度8:  
Correct: 90.82%, Predict selfies valid: 97.96%, error_type
NO                  90.816327
error_atom_seq       5.102041
error_bond_count     2.040816
error_bond_seq       1.020408
error_atom_count     1.020408

# epoch=2
评估长度5: 
Correct: 92.00%, Predict selfies valid: 99.00%, error_type
NO                  91.0
error_bond_count     4.0
error_atom_seq       3.0
error_branch         2.0

评估长度6: 
Correct: 89.00%, Predict selfies valid: 99.00%, error_type
NO                  84.0
error_atom_seq      11.0
error_ring           3.0
error_bond_count     2.0

评估长度7: 
Correct: 80.00%, Predict selfies valid: 99.00%, error_type
NO                  76.0
error_atom_seq      15.0
error_ring           5.0
error_bond_count     3.0
error_branch         1.0

评估长度8: 
Correct: 89.80%, Predict selfies valid: 97.96%, error_type
NO                  89.795918
error_atom_seq       6.122449
error_bond_count     4.081633
 
# epoch=3

评估长度5: 
Correct: 94.00%, Predict selfies valid: 99.00%, error_type
NO                  92.0
error_branch         3.0
error_atom_seq       2.0
error_bond_count     2.0
error_ring           1.0

评估长度6: 
Correct: 92.00%, Predict selfies valid: 99.00%, error_type
NO                  87.0
error_atom_seq       8.0
error_ring           4.0
error_bond_count     1.0

评估长度7: 
Correct: 82.00%, Predict selfies valid: 97.00%, error_type
NO                  78.0
error_atom_seq      12.0
error_ring           7.0
error_bond_count     2.0
error_branch         1.0

评估长度8: 
Correct: 89.80%, Predict selfies valid: 98.98%, error_type
NO                  89.795918
error_atom_seq       9.183673
error_atom_count     1.020408
 
# epoch=4

评估长度5: 
Correct: 95.00%, Predict selfies valid: 100.00%, error_type
NO                  94.0
error_bond_count     3.0
error_branch         2.0
error_atom_seq       1.0

评估长度6: 
Correct: 93.00%, Predict selfies valid: 100.00%, error_type
NO                  86.0
error_atom_seq      10.0
error_ring           3.0
error_bond_count     1.0

评估长度7: 
Correct: 85.00%, Predict selfies valid: 96.00%, error_type
NO                  82.0
error_atom_seq       8.0
error_ring           7.0
error_bond_count     3.0

评估长度8: 
Correct: 83.67%, Predict selfies valid: 98.98%, error_type
NO                  83.673469
error_atom_seq      13.265306
error_atom_count     2.040816
error_bond_seq       1.020408
``` 

评估长度7的SOTA(epoch=4): [31.analyzed.2137.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/31.analyzed.2137.log), 其中:

  - 1个: ring未识别
  - 1个: ring镜像+原子交换
  - 6个: branch错误
  - 3个: ring镜像+键错误
  - 1个: ring相关, 原子交换
  - 3个: ring相关, 键错误

比起未引入镜像时的评估结果 (epoch=4): ring相关的错误, 在之前会更复杂 (需要多次原子交换), 这次会更集中于简单的键错误. 

认为ring镜像数据是有效的 (但之前未引入长度7的训练数据时, 评估ring镜像数据是无效的, 当时并未关注于镜像后的错误难易程度, 之后可能需要进行单变量测试)

# 将训练步骤拆成更多步骤进行尝试

```
步骤1: 长度4普通分子和增强
$(train_dataset_of_length "cot_with_CNO_count/chain_selfies" "4"),\
$(train_dataset_of_length "cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies" "4"),\
$(train_dataset_of_length "cot_with_CNO_count/swap_atom_selfies_from_chain_selfies" "3" "4")

 
步骤2: 长度5普通分子和增强
$(train_dataset_of_length "cot_with_CNO_count/chain_selfies" "5"),\
$(train_dataset_of_length "cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies" "5"),\
$(train_dataset_of_length "cot_with_CNO_count/swap_atom_selfies_from_chain_selfies" "3" "5")

步骤3: 长度5 ring分子和增强
$(train_dataset_of_length "cot_with_CNO_count/ring_selfies" "5" "5"),\
$(train_dataset_of_length "cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies" "5" "5")

步骤4: 长度6普通分子和增强
$(train_dataset_of_length "cot_with_CNO_count/chain_selfies" "6"),\
$(train_dataset_of_length "cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies" "6"),\
$(train_dataset_of_length "cot_with_CNO_count/swap_atom_selfies_from_chain_selfies" "3" "6")

步骤5: 长度6 ring分子和增强
$(train_dataset_of_length "cot_with_CNO_count/ring_selfies" "5" "6"),\
$(train_dataset_of_length "cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies" "5" "6")

步骤6: 长度7普通分子和增强
$(train_dataset_of_length "cot_with_CNO_count/chain_selfies" "7"),\
$(train_dataset_of_length "cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies" "7"),\
$(train_dataset_of_length "cot_with_CNO_count/swap_atom_selfies_from_chain_selfies" "3" "7")

步骤7: 长度7 ring分子和增强
$(train_dataset_of_length "cot_with_CNO_count/ring_selfies" "5" "7"),\
$(train_dataset_of_length "cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies" "5" "7")
``` 

评估结果: 

```
# epoch=1

评估长度5: 
Correct: 89.00%, Predict selfies valid: 97.00%, error_type
NO                  88.0
error_branch         4.0
error_bond_count     3.0
error_ring           3.0
error_atom_seq       1.0
error_atom_count     1.0

评估长度6: 
Correct: 83.00%, Predict selfies valid: 94.00%, error_type
NO                  82.0
error_atom_seq       9.0
error_ring           4.0
error_bond_count     2.0
                     2.0
error_bond_seq       1.0

评估长度7: 
Correct: 75.00%, Predict selfies valid: 90.00%, error_type
NO                  75.0
error_branch         8.0
error_atom_seq       7.0
error_ring           4.0
                     4.0
error_bond_seq       1.0
error_bond_count     1.0

评估长度8: 
Correct: 86.73%, Predict selfies valid: 97.96%, error_type
NO                  86.734694
error_atom_seq       7.142857
error_bond_count     3.061224
error_bond_seq       2.040816
error_atom_count     1.020408

# epoch=2

评估长度5: 
Correct: 92.00%, Predict selfies valid: 95.00%, error_type
NO                  90.0
error_ring           5.0
error_branch         2.0
error_bond_count     2.0
error_atom_seq       1.0

评估长度6: 
Correct: 83.00%, Predict selfies valid: 97.00%, error_type
NO                  82.0
error_atom_seq       9.0
error_ring           5.0
error_bond_count     2.0
error_branch         2.0

评估长度7: 
Correct: 78.00%, Predict selfies valid: 91.00%, error_type
NO                74.0
error_atom_seq    11.0
error_ring         8.0
error_branch       7.0

评估长度8: 
Correct: 88.78%, Predict selfies valid: 97.96%, error_type
NO                  88.775510
error_atom_seq       7.142857
error_atom_count     3.061224
error_bond_count     1.020408

# epoch=3

评估长度5: 
Correct: 91.00%, Predict selfies valid: 96.00%, error_type
NO                  90.0
error_ring           7.0
error_atom_seq       2.0
error_bond_count     1.0

评估长度6: 
Correct: 89.00%, Predict selfies valid: 94.00%, error_type
NO                  85.0
error_atom_seq       7.0
error_ring           4.0
error_branch         3.0
error_bond_count     1.0

评估长度7: 
Correct: 80.00%, Predict selfies valid: 92.00%, error_type
NO                77.0
error_atom_seq    12.0
error_ring         8.0
error_branch       3.0

评估长度8: 
Correct: 88.78%, Predict selfies valid: 100.00%, error_type
NO                  88.775510
error_atom_seq       9.183673
error_bond_count     1.020408
error_atom_count     1.020408

# epoch=4

评估长度5: 
Correct: 91.00%, Predict selfies valid: 98.00%, error_type
NO                  90.0
error_ring           6.0
error_bond_seq       1.0
error_atom_seq       1.0
error_branch         1.0
error_bond_count     1.0

评估长度6: 
Correct: 87.00%, Predict selfies valid: 94.00%, error_type
NO                  84.0
error_atom_seq       7.0
error_ring           4.0
error_branch         2.0
error_bond_count     2.0
error_atom_count     1.0

评估长度7: 
Correct: 79.00%, Predict selfies valid: 91.00%, error_type
NO                  76.0
error_atom_seq       8.0
error_ring           7.0
error_branch         5.0
error_atom_count     2.0
error_bond_count     1.0
error_other          1.0

评估长度8: 
Correct: 78.57%, Predict selfies valid: 100.00%, error_type
NO                  78.571429
error_atom_seq      16.326531
error_bond_count     2.040816
error_bond_seq       1.020408
error_atom_count     1.020408
error_other          1.020408
``` 

整体效果是不如 步骤较少的训练的. 认为: 如果对比的数据不放在一起, 那么不容易进行对比学习, 或者是因为学习率不够

额外, 评估一下长度7评估的SOTA的错误类型: [12.analyzed.2222.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/12.analyzed.2222.log)

ring的错误, 大部分是ring长度错误, 学习效果比较差

# 增加针对ring和ring镜像数据的bond增强数据

commit: 4c562848232ffae94dc6c29d9fdadd81f7c5a800

针对ring和ring的镜像selfies, 随机生成另一组bond配置, 作为增强数据

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_7.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_7.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/raw/length_5.train.json:18
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_5.train.json:18
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/raw/length_7.train.json:258
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_7.train.json:258
``` 

注意: randomize_bond_from_ring_selfies_and_mirror 中数据量是ring的2倍 (里面还包括了ring的镜像数据的增强)

训练效果: 

```
# epoch=1

评估长度5: 
Correct: 90.00%, Predict selfies valid: 96.00%, error_type
NO                  89.0
error_ring           4.0
error_bond_count     3.0
error_atom_seq       3.0
error_branch         1.0

评估长度6: 
Correct: 91.00%, Predict selfies valid: 97.00%, error_type
NO                  85.0
error_atom_seq       8.0
error_branch         3.0
error_bond_count     2.0
error_bond_seq       1.0
error_ring           1.0

评估长度7: 
Correct: 82.00%, Predict selfies valid: 93.00%, error_type
NO                  76.0
error_atom_seq      15.0
error_ring           5.0
error_branch         2.0
error_atom_count     1.0
error_bond_count     1.0

评估长度8: 
Correct: 88.78%, Predict selfies valid: 98.98%, error_type
NO                  88.775510
error_atom_seq       9.183673
error_bond_seq       1.020408
error_atom_count     1.020408

# epoch=2

评估长度5: 
Correct: 94.00%, Predict selfies valid: 97.00%, error_type
NO                  93.0
error_atom_seq       3.0
error_ring           2.0
error_branch         1.0
error_bond_count     1.0

评估长度6: 
Correct: 94.00%, Predict selfies valid: 98.00%, error_type
NO                  88.0
error_atom_seq       6.0
error_ring           3.0
error_bond_count     1.0
error_bond_seq       1.0
error_branch         1.0

评估长度7: 
Correct: 86.00%, Predict selfies valid: 95.00%, error_type
NO                  80.0
error_ring           7.0
error_atom_seq       6.0
error_bond_count     3.0
error_atom_count     2.0
error_branch         1.0
                     1.0

评估长度8: 
Correct: 93.88%, Predict selfies valid: 100.00%, error_type
NO                  93.877551
error_atom_seq       5.102041
error_bond_count     1.020408

# epoch=3

评估长度5: 
Correct: 95.00%, Predict selfies valid: 98.00%, error_type
NO                94.0
error_ring         4.0
                   1.0
error_atom_seq     1.0

评估长度6: 
Correct: 95.00%, Predict selfies valid: 99.00%, error_type
NO                90.0
error_atom_seq     4.0
error_branch       3.0
error_ring         1.0
error_bond_seq     1.0
error_other        1.0

评估长度7: 
Correct: 87.00%, Predict selfies valid: 96.00%, error_type
NO                  84.0
error_atom_seq       9.0
error_ring           4.0
error_bond_count     1.0
error_atom_count     1.0
error_branch         1.0

评估长度8: 
Correct: 86.73%, Predict selfies valid: 98.98%, error_type
NO                86.734694
error_atom_seq    12.244898
error_bond_seq     1.020408

# epoch=4

评估长度5: 
Correct: 96.00%, Predict selfies valid: 97.00%, error_type
NO                96.0
error_ring         2.0
error_branch       1.0
error_atom_seq     1.0

评估长度6: 
Correct: 99.00%, Predict selfies valid: 97.00%, error_type
NO                93.0
error_atom_seq     4.0
error_branch       3.0

评估长度7: 
Correct: 87.00%, Predict selfies valid: 95.00%, error_type
NO                  79.0
error_atom_seq      13.0
error_ring           3.0
error_branch         3.0
error_bond_count     1.0
error_bond_seq       1.0

评估长度8: 
Correct: 94.90%, Predict selfies valid: 98.98%, error_type
NO                  94.897959
error_atom_seq       3.061224
error_bond_seq       1.020408
error_bond_count     1.020408
``` 

长度7的识别率大幅提升. 分析SOTA(epoch=3)的错误case: [11.analyzed.1420.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/11.analyzed.1420.log), 其中:

  - 8个: branch错误
  - 3个: ring相关:
    - 1个: 其余部分进行镜像 (环识别错误)
    - 1个: 其余部分原子交换+键错误 (环识别正确, 在环内做原子交换+键错误)
    - 1个: 其余部分原子交换 (环识别正确, 在环内做原子交换)
  - 1个: ring镜像后, 原子交换+键错误 (环识别正确, 在环内做原子交换+键错误)
  - 1个: 普通分子, 镜像后, 键错误

下一步调优branch错误, 分析以上日志中的branch错误分类: [11.analyzed.1518.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/11.analyzed.1518.log) , 其中:

  - branch错误, 起点设为1 + 原子交换  
branch错误, 起点设为1 + 除去Ring  
branch错误, 识别不出branch, 没有规律  
branch错误, 将识别结果的起点设为3, 得到与GT相似分子  
branch错误, 将识别结果镜像, 得到GT去除分支操作符的分子  
branch错误, 将识别结果的起点设为3, 得到GT去除分支操作符的分子  
branch错误, 起点设为2 + 除去最后两个C  
branch错误, 起点设为1, 得到与GT相似分子

看起来将branch分子进行增强, 需要不仅使用镜像, 还需要设置不同的起点来形成分子, 可能会有效果

将长度6/7/8的branch生成个数扩大 (100 -> 150), 然后进行增强

# 将长度6/7/8的branch生成个数扩大 (100 -> 150), 然后进行增强

commit: f71c9d70d922813cef1f9351c55c5bc78d1894c7

对于每一个branch selfies, 随机选一个起始位置, 生成新的selfies

注意: 随机起始位置后, selfies的长度可能会变, 可能变长或者变短. (长度5的branch selfies, 变化后可能是长度4或7等)

数据量: 

```
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_2.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_3.train.json:36
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_4.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/raw/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/chain_selfies/enhanced_by_image/length_7.train.json:380
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:28
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:89
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_7.train.json:371
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_7.train.json:120
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_7.train.json:120
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_start_point_from_branch_selfies/raw/length_3.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_start_point_from_branch_selfies/enhanced_by_image/length_3.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_start_point_from_branch_selfies/raw/length_4.train.json:45
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_start_point_from_branch_selfies/enhanced_by_image/length_4.train.json:45
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_start_point_from_branch_selfies/raw/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_start_point_from_branch_selfies/enhanced_by_image/length_5.train.json:35
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_start_point_from_branch_selfies/raw/length_6.train.json:57
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_start_point_from_branch_selfies/enhanced_by_image/length_6.train.json:57
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_start_point_from_branch_selfies/raw/length_7.train.json:59
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_start_point_from_branch_selfies/enhanced_by_image/length_7.train.json:59
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_mirror_selfies/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:16
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:80
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_7.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/raw/length_5.train.json:18
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_5.train.json:18
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/raw/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_6.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/raw/length_7.train.json:258
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/randomize_bond_from_ring_selfies_and_mirror/enhanced_by_image/length_7.train.json:258 
``` 

训练时间: 1个epoch 近一个小时

训练效果: 

```
# epoch=1

评估长度5: 
Correct: 93.00%, Predict selfies valid: 96.00%, error_type
NO                  88.0
error_atom_seq       6.0
error_bond_seq       2.0
                     1.0
error_bond_count     1.0
error_ring           1.0
error_other          1.0

评估长度6: 
Correct: 93.00%, Predict selfies valid: 97.00%, error_type
NO                  88.0
error_atom_seq       6.0
error_bond_count     2.0
error_branch         2.0
error_ring           1.0
error_bond_seq       1.0

评估长度7: 
Correct: 79.00%, Predict selfies valid: 95.00%, error_type
NO                  77.0
error_atom_seq      14.0
error_ring           7.0
error_bond_count     2.0

评估长度8: 
Correct: 83.67%, Predict selfies valid: 97.96%, error_type
NO                  83.673469
error_atom_seq      11.224490
error_bond_seq       4.081633
error_atom_count     1.020408

# epoch=2

评估长度5: 
Correct: 97.00%, Predict selfies valid: 97.00%, error_type
NO                95.0
error_ring         3.0
error_atom_seq     2.0

评估长度6: 
Correct: 94.00%, Predict selfies valid: 97.00%, error_type
NO                  91.0
error_atom_seq       3.0
error_bond_count     2.0
error_branch         2.0
error_ring           1.0
error_bond_seq       1.0

评估长度7: 
Correct: 90.00%, Predict selfies valid: 96.00%, error_type
NO                83.0
error_atom_seq    10.0
error_ring         6.0
                   1.0

评估长度8: 
Correct: 91.84%, Predict selfies valid: 98.98%, error_type
NO                  90.816327
error_atom_seq       6.122449
error_bond_count     2.040816
error_atom_count     1.020408 
 

# epoch=3

评估长度5: 
Correct: 97.00%, Predict selfies valid: 99.00%, error_type
NO                95.0
error_ring         4.0
error_atom_seq     1.0

评估长度6: 
Correct: 97.00%, Predict selfies valid: 98.00%, error_type
NO                  90.0
error_atom_seq       5.0
error_ring           3.0
error_bond_count     1.0
error_branch         1.0

评估长度7: 
Correct: 90.00%, Predict selfies valid: 95.00%, error_type
NO                  81.0
error_atom_seq      11.0
error_ring           3.0
error_branch         3.0
error_bond_count     1.0
error_other          1.0

评估长度8: 
Correct: 82.65%, Predict selfies valid: 97.96%, error_type
NO                82.653061
error_atom_seq    16.326531
error_bond_seq     1.020408

# epoch=4

评估长度5: 
Correct: 95.00%, Predict selfies valid: 98.00%, error_type
NO                  95.0
error_bond_count     2.0
error_ring           2.0
error_branch         1.0

评估长度6: 
Correct: 98.00%, Predict selfies valid: 97.00%, error_type
NO                91.0
error_atom_seq     5.0
error_branch       3.0
error_ring         1.0

评估长度7: 
Correct: 93.00%, Predict selfies valid: 97.00%, error_type
NO                  84.0
error_atom_seq      10.0
error_ring           4.0
error_bond_count     1.0
error_branch         1.0

评估长度8: 
Correct: 94.90%, Predict selfies valid: 100.00%, error_type
NO                94.897959
error_atom_seq     4.081633
error_bond_seq     1.020408
``` 

epoch=4的效果最好, 但时间太长, 先分析其他epoch

已经上升到90%, 分析长度7的错误case (epoch=2): [7.analyzed.2316.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/7.analyzed.2316.log), 其中:

  - 2个: 更换起点后, (可能是被提示词中的原子数制约), 多了原子
  - 3个: 没有规律
  - 4个: 更换起点后, 分支/环内乱序, 或者键错误

分析长度7的错误case (epoch=3): [11.analyzed.2336.log](/assets/01KJBZQT2DD1YNRJ2J49PP9MN0/11.analyzed.2336.log) , 其中:

  - 1个: 没有规律
  - 1个: 原子交换
  - 2个: 键错误
  - 2个: 更换起点后, ring需要位移
  - 2个: 更换起点后, 分支/环内乱序
  - 1个: 更换起点后, (可能是被提示词中的原子数制约), 多了原子
  - 1个: 更换起点后, branch长度错误, 其他正确

# 结论

通过对ring和branch的数据增强, 增强了长度7 在"训练数据包括长度7数据"的情况下 的准确率

不用测试长度7 在"训练数据不包括长度7数据"的情况下 的准确率, 因为ring和branch的一些问题只有长度7时才出现, 无法通过长度5/6进行推断.
