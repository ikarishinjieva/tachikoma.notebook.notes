---
title: 20250118 - 使用ms-swift对Qwen2-VL进行微调 [19] - 调优长度7的效果[2]
confluence_page_id: 3343884
created_at: 2025-01-18T04:56:41+00:00
updated_at: 2025-01-22T15:44:11+00:00
---

# 前序

重新整理了各个脚本, 形成项目: <https://github.com/ikarishinjieva/mol-ai>

需要重新建立基线 (数据重新生成过), 分析错误case, 进行数据增强

# 基线

  1. 开始训练: 
     1. 识别任务, 链式生成的selfies, 提示词使用COT, 并指定C/N/O的原子数: 长度1-3数据以及增强图形 (每种最多取100个), 长度4数据以及增强图形 (取全量)
     2. 识别任务, 上一步的增强数据 (镜像selfies): 长度1-3数据以及增强图形 (每种最多取100个), 长度4数据以及增强图形 (取全量)
  2. 继续训练:
     1. 识别任务, 图像(链式生成的selfies), 提示词(使用COT, 并指定C/N/O的原子数): 长度1-4数据以及增强图形 (每种最多取100个), 长度5数据以及增强图形 (取全量)
     2. 识别任务, 图像(上一步的增强数据, 镜像selfies), 提示词(同前): 长度1-4数据以及增强图形 (每种最多取100个), 长度5数据以及增强图形 (取全量)
     3. 识别任务, 图像(ring的长度5的图像), 提示词(同前): 长度5的数据以及增强图形 (取全量)
     4. 识别任务, 图像(上一步的增强数据, 随机生成等价的selfies), 提示词(同前): 长度5的数据 (是上一步数据的大约1.5倍) 以及增强图形 (取全量)
  3. 继续训练:
     1. 识别任务, 图像(链式生成的selfies), 提示词(使用COT, 并指定C/N/O的原子数): 长度1-5数据以及增强图形 (每种最多取100个), 长度6数据以及增强图形 (取全量)
     2. 识别任务, 图像(上一步的增强数据, 镜像selfies), 提示词(同前): 长度1-5数据以及增强图形 (每种最多取100个), 长度6数据以及增强图形 (取全量)
     3. 识别任务, 图像(ring的长度5的图像), 提示词(同前): 长度5数据以及增强图形 (每种最多取100个), 长度6的数据以及增强图形 (取全量)
     4. 识别任务, 图像(上一步的增强数据, 随机生成等价的selfies, 是上一步数据的大约1.5倍), 提示词(同前): 长度5的数据以及增强图形 (每种最多取100个), 长度6的数据以及增强图形 (取全量)

评估结果: 

```
# epoch=1 
 
长度5:
Correct: 84.00%, Predict selfies valid: 98.00%, error_type
NO                  80.0
error_bond_count     8.0
error_ring           4.0
error_atom_seq       3.0
error_branch         2.0
error_bond_seq       2.0
error_atom_count     1.0

长度6:
Correct: 75.00%, Predict selfies valid: 92.00%, error_type
NO                  74.0
error_bond_count     6.0
error_atom_seq       6.0
error_bond_seq       5.0
error_ring           4.0
error_branch         3.0
error_atom_count     1.0
error_other          1.0

长度7:
Correct: 55.00%, Predict selfies valid: 86.00%, error_type
NO                  53.0
error_atom_seq      14.0
error_bond_count     9.0
error_ring           8.0
error_branch         8.0
error_bond_seq       6.0
error_other          2.0

# epoch=2
 
长度5:
Correct: 88.00%, Predict selfies valid: 98.00%, error_type
NO                  85.0
error_ring           5.0
error_bond_count     5.0
error_atom_seq       4.0
error_branch         1.0

长度6:
Correct: 79.00%, Predict selfies valid: 94.00%, error_type
NO                  77.0
error_bond_count     9.0
error_ring           6.0
error_atom_seq       5.0
error_bond_seq       2.0
error_branch         1.0

长度7:
Correct: 65.00%, Predict selfies valid: 90.00%, error_type
NO                  63.0
error_atom_seq      12.0
error_branch         9.0
error_ring           6.0
error_atom_count     4.0
error_bond_seq       3.0
error_bond_count     3.0

# epoch=3

长度5:
Correct: 85.00%, Predict selfies valid: 98.00%, error_type
NO                  84.0
error_bond_count     7.0
error_ring           6.0
error_bond_seq       2.0
error_atom_seq       1.0

长度6:
Correct: 85.00%, Predict selfies valid: 96.00%, error_type
NO                  82.0
error_atom_seq       6.0
error_bond_count     4.0
error_ring           4.0
error_branch         3.0
error_atom_count     1.0

长度7:
Correct: 71.00%, Predict selfies valid: 93.00%, error_type
NO                  69.0
error_branch         8.0
error_ring           7.0
error_atom_seq       6.0
error_bond_count     5.0
error_atom_count     3.0
error_other          1.0
error_bond_seq       1.0

# epoch=4
 
长度5:
Correct: 88.00%, Predict selfies valid: 100.00%, error_type
NO                  85.0
error_bond_count     8.0
error_branch         3.0
error_ring           3.0
error_bond_seq       1.0

长度6:
Correct: 84.00%, Predict selfies valid: 98.00%, error_type
NO                  81.0
error_bond_count     6.0
error_atom_seq       5.0
error_ring           4.0
error_branch         2.0
                     1.0
error_atom_count     1.0

长度7:
Correct: 70.00%, Predict selfies valid: 93.00%, error_type
NO                  68.0
error_atom_seq      13.0
error_branch         9.0
error_ring           7.0
error_bond_count     1.0
error_other          1.0
error_bond_seq       1.0

# epoch=6

长度5:
Correct: 85.00%, Predict selfies valid: 98.00%, error_type
NO                  83.0
error_bond_count    10.0
error_ring           5.0
error_branch         1.0
error_atom_seq       1.0

长度6:
Correct: 89.00%, Predict selfies valid: 98.00%, error_type
NO                  84.0
error_atom_seq       5.0
error_branch         4.0
error_ring           3.0
error_bond_count     2.0
error_atom_count     1.0
error_bond_seq       1.0

长度7:
Correct: 70.00%, Predict selfies valid: 89.00%, error_type
NO                  68.0
error_atom_seq      11.0
error_branch        11.0
error_ring           4.0
error_atom_count     2.0
error_bond_count     2.0
error_bond_seq       1.0
error_other          1.0
 

# epoch=8

长度5:
Correct: 90.00%, Predict selfies valid: 100.00%, error_type
NO                  86.0
error_bond_count     6.0
error_branch         4.0
error_ring           2.0
error_atom_seq       1.0
error_bond_seq       1.0

长度6:
Correct: 87.00%, Predict selfies valid: 96.00%, error_type
NO                  84.0
error_ring           7.0
error_atom_seq       5.0
error_branch         2.0
error_bond_count     2.0

长度7:
Correct: 66.00%, Predict selfies valid: 90.00%, error_type
NO                  64.0
error_atom_seq      13.0
error_branch        12.0
error_ring           4.0
error_bond_count     3.0
error_bond_seq       2.0
error_other          2.0

``` 

epoch=3/4/6 对长度7的准确率高于70%, 对其中错误进行分析, 评估日志: 

  - epoch=3: [9.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/9.log): ring/branch错误占比较大, 需要增加ring环节的epoch配比? 
  - epoch=4: [12.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/12.log)
    - 训练数据中, 没有"[Ring1][Branch1]"这个组合
    - 训练数据中, 没有单独的"[Branch1]"的训练数据
    - 其中有10+组数据, 推理的结果和GT之间, 可以通过相邻原子交换或整体轮换来转化: [12.analyzed.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/12.analyzed.log)
  - epoch=6: [15 (1).log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/15%20%281%29.log)
    - 其中有8组数据, 推理的结果和GT之间, 可以通过相邻原子交换或整体轮换来转化: [15.analyzed (1).log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/15.analyzed%20%281%29.log)

# 增加[Ring1][Branch1]的训练数据

生成ring数据是, 发现只有原子数7开始的分子, 才能有[Ring1][Branch1]结构

所以现在先忽略"[Ring1][Branch1]"的问题

# 生成原子交换的增强数据

代码commit: 4b165c89017582208a37e13eff1cdcb7f921af5c

增强数据的方法: images/swap_atom_selfies_from_chain_selfies/generate_images.ipynb

  1. 根据chain_selfies(链式生成的selfies)中, 对于每一个selfies
     1. 随机选取一个原子与相邻的原子交换位置, 
     2. 然后对于每一个原子随机配置单键/双键/三键, 
     3. 最后检查其合法性

训练步骤:

  1. 开始训练: 
     1. 识别任务, 链式生成的selfies, 提示词使用COT, 并指定C/N/O的原子数: 长度1-3数据以及增强图形 (每种最多取100个), 长度4数据以及增强图形 (取全量)
     2. 识别任务, 上一步的增强数据 (镜像selfies): 长度1-3数据以及增强图形 (每种最多取100个), 长度4数据以及增强图形 (取全量)
     3. 识别任务, 上一步的增强数据 (交换原子位置): 长度3数据以及增强图形 (每种最多取100个), 长度4数据以及增强图形 (取全量)
  2. 继续训练:
     1. 识别任务, 图像(链式生成的selfies), 提示词(使用COT, 并指定C/N/O的原子数): 长度1-4数据以及增强图形 (每种最多取100个), 长度5数据以及增强图形 (取全量)
     2. 识别任务, 图像(上一步的增强数据, 镜像selfies), 提示词(同前): 长度1-4数据以及增强图形 (每种最多取100个), 长度5数据以及增强图形 (取全量)
     3. 识别任务, 上一步的增强数据 (交换原子位置): 长度3-4数据以及增强图形 (每种最多取100个), 长度5数据以及增强图形 (取全量)
     4. 识别任务, 图像(ring的长度5的图像), 提示词(同前): 长度5的数据以及增强图形 (取全量)
     5. 识别任务, 图像(上一步的增强数据, 随机生成等价的selfies), 提示词(同前): 长度5的数据 (是上一步数据的大约1.5倍) 以及增强图形 (取全量)
  3. 继续训练:
     1. 识别任务, 图像(链式生成的selfies), 提示词(使用COT, 并指定C/N/O的原子数): 长度1-5数据以及增强图形 (每种最多取100个), 长度6数据以及增强图形 (取全量)
     2. 识别任务, 图像(上一步的增强数据, 镜像selfies), 提示词(同前): 长度1-5数据以及增强图形 (每种最多取100个), 长度6数据以及增强图形 (取全量)
     3. 识别任务, 上一步的增强数据 (交换原子位置): 长度3-5数据以及增强图形 (每种最多取100个), 长度6数据以及增强图形 (取全量)
     4. 识别任务, 图像(ring的长度5的图像), 提示词(同前): 长度5数据以及增强图形 (每种最多取100个), 长度6的数据以及增强图形 (取全量)
     5. 识别任务, 图像(上一步的增强数据, 随机生成等价的selfies, 是上一步数据的大约1.5倍), 提示词(同前): 长度5的数据以及增强图形 (每种最多取100个), 长度6的数据以及增强图形 (取全量)

  

评估结果: 

```
# epoch=1 
 
长度5:
Correct: 93.00%, Predict selfies valid: 99.00%, error_type
NO                  91.0
error_atom_seq       5.0
error_branch         2.0
error_bond_count     2.0

长度6:
Correct: 82.00%, Predict selfies valid: 93.00%, error_type
NO                  82.0
error_atom_seq       8.0
error_branch         4.0
error_ring           3.0
error_bond_count     2.0
error_bond_seq       1.0

长度7:
Correct: 63.00%, Predict selfies valid: 92.00%, error_type
NO                  58.0
error_atom_seq      13.0
error_branch         9.0
error_bond_count     7.0
error_ring           7.0
error_bond_seq       4.0
error_atom_count     2.0

# epoch=2
 
长度5:
Correct: 93.00%, Predict selfies valid: 98.00%, error_type
NO                  93.0
error_atom_seq       2.0
error_bond_count     2.0
error_branch         1.0
error_bond_seq       1.0
error_ring           1.0

长度6:
Correct: 84.00%, Predict selfies valid: 93.00%, error_type
NO                  83.0
error_ring           6.0
error_atom_seq       4.0
error_branch         2.0
error_bond_count     2.0
error_atom_count     2.0
error_bond_seq       1.0

长度7:
Correct: 71.00%, Predict selfies valid: 88.00%, error_type
NO                  68.0
error_branch         9.0
error_atom_seq       9.0
error_ring           5.0
error_bond_seq       4.0
error_atom_count     3.0
error_bond_count     1.0
error_other          1.0

# epoch=3

长度5:
Correct: 98.00%, Predict selfies valid: 99.00%, error_type
NO                94.0
error_atom_seq     4.0
error_ring         2.0

长度6:
Correct: 86.00%, Predict selfies valid: 96.00%, error_type
NO                83.0
error_atom_seq     6.0
error_ring         5.0
error_bond_seq     4.0
error_branch       2.0

长度7:
Correct: 69.00%, Predict selfies valid: 89.00%, error_type
NO                  65.0
error_atom_seq      16.0
error_ring           8.0
error_branch         5.0
error_bond_count     4.0
error_bond_seq       1.0
error_atom_count     1.0

# epoch=4
 
长度5:
Correct: 94.00%, Predict selfies valid: 98.00%, error_type
NO                  92.0
error_atom_seq       3.0
error_bond_count     2.0
error_ring           1.0
error_bond_seq       1.0
error_branch         1.0

长度6:
Correct: 90.00%, Predict selfies valid: 95.00%, error_type
NO                86.0
error_ring         4.0
error_branch       4.0
error_atom_seq     4.0
error_bond_seq     2.0

长度7:
Correct: 72.00%, Predict selfies valid: 92.00%, error_type
NO                  68.0
error_branch        10.0
error_atom_seq       9.0
error_ring           7.0
error_bond_seq       3.0
error_bond_count     2.0
error_atom_count     1.0

# epoch=6

长度5:
Correct: 96.00%, Predict selfies valid: 99.00%, error_type
NO                  94.0
error_atom_seq       3.0
error_ring           2.0
error_bond_count     1.0

长度6:
Correct: 87.00%, Predict selfies valid: 94.00%, error_type
NO                85.0
error_atom_seq     6.0
error_ring         4.0
error_branch       4.0
error_bond_seq     1.0

长度7:
Correct: 74.00%, Predict selfies valid: 89.00%, error_type
NO                  71.0
error_atom_seq      13.0
error_branch         9.0
error_ring           3.0
error_bond_count     2.0
error_atom_count     2.0

# epoch=8

长度5:
Correct: 95.00%, Predict selfies valid: 97.00%, error_type
NO                  93.0
error_bond_count     2.0
error_branch         2.0
error_bond_seq       2.0
error_atom_seq       1.0

长度6:
Correct: 85.00%, Predict selfies valid: 96.00%, error_type
NO                  82.0
error_atom_seq       7.0
error_ring           4.0
error_branch         4.0
error_bond_count     2.0
error_bond_seq       1.0

长度7:
Correct: 71.00%, Predict selfies valid: 92.00%, error_type
NO                  65.0
error_atom_seq      14.0
error_branch         9.0
error_ring           4.0
error_bond_count     4.0
error_bond_seq       2.0
error_atom_count     2.0

``` 

与基线对比: 

  - 对长度5的正确率大幅上升 (均超过93%)
  - 对长度6/7的正确率没有明显提升
  - SOTA出现在epoch=6 (74%), 分析SOTA的错误分布: [15.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/15.log)
    - 分析结果: [15.analyzed.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/15.analyzed.log)
    - ~~其中: 有13个error_atom_seq错误, 8个和ring相关 (通过ring交换/轮换/镜像, 可以修正)~~
    - ~~在错误的case中, 有6个error_branch错误, 其中有4个错误, 是没有识别出Branch1~~
    - ~~检查训练数据, 包括[Branch1]但不是跟[Ring1]联用的case, 仅有8个, 需要单独为branch增加训练数据~~
    - 在理解了branch机制之后 (见后续章节), 重新评估这份数据, 有8个case是[Branch]结构并未被识别出来
  - error_atom_seq错误中, 非ring相关的错误下降一半以上, 也就是说增强数据有效, 但错误向ring相关的case转移, 所以整体错误率没有改善

  

# 增强数据: 单独使用[Branch]的分子

代码commit: 988fead103d2259e4deb8891fcbb5acc25b758ab

增强数据的方法: images/branch_selfies_from_chain_selfies/generate_images.ipynb

  1. 根据chain_selfies(链式生成的selfies)中, 对于每一个selfies
     1. 随机选取一个原子, 替换成"[branch1]"
     2. 检查其合法性, 如果合法, 则记录下来

以上增强方法的数据量较少, 数据量统计: 

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
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_3.train.json:29
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_3.train.json:29
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_4.train.json:92
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_4.train.json:92
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:100
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/raw/length_6.train.json:318
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/swap_atom_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:318
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/raw/length_5.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:10
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/raw/length_6.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:13
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:13
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:24
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:24
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_5.train.json:19
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_5.train.json:19
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_6.train.json:40
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_6.train.json:40

``` 

训练步骤:

  1. 开始训练: 
     1. 识别任务, 链式生成的selfies, 提示词使用COT, 并指定C/N/O的原子数: 长度1-3数据以及增强图形 (每种最多取100个), 长度4数据以及增强图形 (取全量)
     2. 识别任务, 上一步的增强数据 (镜像selfies): 长度1-3数据以及增强图形 (每种最多取100个), 长度4数据以及增强图形 (取全量)
     3. 识别任务, 上一步的增强数据 (交换原子位置): 长度3数据以及增强图形 (每种最多取100个), 长度4数据以及增强图形 (取全量)
  2. 继续训练:
     1. 识别任务, 图像(链式生成的selfies), 提示词(使用COT, 并指定C/N/O的原子数): 长度1-4数据以及增强图形 (每种最多取100个), 长度5数据以及增强图形 (取全量)
     2. 识别任务, 图像(上一步的增强数据, 镜像selfies), 提示词(同前): 长度1-4数据以及增强图形 (每种最多取100个), 长度5数据以及增强图形 (取全量)
     3. 识别任务, 上一步的增强数据 (交换原子位置): 长度3-4数据以及增强图形 (每种最多取100个), 长度5数据以及增强图形 (取全量)
     4. 识别任务, 上一步的增强数据 (生成[branch]): 长度5数据以及增强图形 (取全量)  

     5. 识别任务, 图像(ring的长度5的图像), 提示词(同前): 长度5的数据以及增强图形 (取全量)
     6. 识别任务, 图像(上一步的增强数据, 随机生成等价的selfies), 提示词(同前): 长度5的数据 (是上一步数据的大约1.5倍) 以及增强图形 (取全量)
  3. 继续训练:
     1. 识别任务, 图像(链式生成的selfies), 提示词(使用COT, 并指定C/N/O的原子数): 长度1-5数据以及增强图形 (每种最多取100个), 长度6数据以及增强图形 (取全量)
     2. 识别任务, 图像(上一步的增强数据, 镜像selfies), 提示词(同前): 长度1-5数据以及增强图形 (每种最多取100个), 长度6数据以及增强图形 (取全量)
     3. 识别任务, 上一步的增强数据 (交换原子位置): 长度3-5数据以及增强图形 (每种最多取100个), 长度6数据以及增强图形 (取全量)
     4. 识别任务, 上一步的增强数据 (生成[branch]): 长度5数据以及增强图形 (每种最多取100个), 长度6数据以及增强图形 (取全量)  

     5. 识别任务, 图像(ring的长度5的图像), 提示词(同前): 长度5数据以及增强图形 (每种最多取100个), 长度6的数据以及增强图形 (取全量)
     6. 识别任务, 图像(上一步的增强数据, 随机生成等价的selfies, 是上一步数据的大约1.5倍), 提示词(同前): 长度5的数据以及增强图形 (每种最多取100个), 长度6的数据以及增强图形 (取全量)

  

训练效果: 

```
# epoch=1
 
评估长度5: 
Correct: 81.00%, Predict selfies valid: 96.00%, error_type
NO                  79.0
error_atom_seq      13.0
error_bond_count     5.0
error_ring           2.0
error_bond_seq       1.0
Name: proportion, dtype: float64

评估长度6:
Correct: 60.00%, Predict selfies valid: 96.00%, error_type
NO                  60.0
error_atom_seq      25.0
error_bond_count     6.0
error_ring           4.0
error_branch         3.0
error_other          1.0
error_bond_seq       1.0

评估长度7:
Correct: 43.00%, Predict selfies valid: 88.00%, error_type
error_atom_seq      41.0
NO                  39.0
error_branch         9.0
error_bond_count     6.0
error_ring           3.0
error_bond_seq       2.0

# epoch=2
 
评估长度5: 
Correct: 80.00%, Predict selfies valid: 99.00%, error_type
NO                  77.0
error_bond_count    13.0
error_atom_seq       5.0
error_ring           3.0
error_other          1.0
error_bond_seq       1.0

评估长度6:
Correct: 66.00%, Predict selfies valid: 96.00%, error_type
NO                  62.0
error_atom_seq      23.0
error_bond_count     8.0
error_ring           4.0
error_branch         2.0
error_atom_count     1.0

评估长度7:
Correct: 44.00%, Predict selfies valid: 90.00%, error_type
NO                  41.0
error_atom_seq      33.0
error_bond_count     8.0
error_branch         7.0
error_bond_seq       5.0
error_ring           4.0
error_atom_count     2.0
``` 

很奇怪: 增加"[branch1]"的分子数并不多, 但长度5/6/7的准确率都大幅下降

反思: 之前重新跑过 swap_atom_selfies_from_chain_selfies 的图片生成, 但没有跑过dataset的生成, 导致json和图片不一致, 调整以后重新跑

  

```
# epoch=1
 
评估长度5: 
Correct: 98.00%, Predict selfies valid: 97.00%, error_type
NO                  97.0
error_atom_seq       2.0
error_bond_count     1.0

评估长度6:
Correct: 83.00%, Predict selfies valid: 96.00%, error_type
NO                  81.0
error_ring           8.0
error_branch         4.0
error_bond_count     3.0
error_atom_seq       2.0
error_atom_count     1.0
error_bond_seq       1.0

评估长度7:
Correct: 62.00%, Predict selfies valid: 86.00%, error_type
NO                  61.0
error_atom_seq      20.0
error_ring           9.0
error_branch         5.0
error_atom_count     2.0
error_bond_seq       2.0
error_bond_count     1.0

# epoch=2
 
评估长度5: 
Correct: 96.00%, Predict selfies valid: 100.00%, error_type
NO                  95.0
error_bond_count     3.0
error_atom_seq       2.0

评估长度6:
Correct: 83.00%, Predict selfies valid: 97.00%, error_type
NO                  81.0
error_ring           8.0
error_atom_seq       5.0
error_bond_count     4.0
error_branch         2.0

评估长度7:
Correct: 70.00%, Predict selfies valid: 90.00%, error_type
NO                  66.0
error_atom_seq      13.0
error_ring           7.0
error_branch         7.0
error_bond_seq       3.0
error_atom_count     2.0
error_bond_count     2.0
# epoch=3
 
评估长度5: 
Correct: 98.00%, Predict selfies valid: 99.00%, error_type
NO                98.0
error_ring         1.0
error_bond_seq     1.0

评估长度6:
Correct: 87.00%, Predict selfies valid: 96.00%, error_type
NO                  85.0
error_ring           7.0
error_atom_seq       6.0
error_branch         1.0
error_bond_count     1.0

评估长度7:
Correct: 68.00%, Predict selfies valid: 91.00%, error_type
NO                  67.0
error_atom_seq      15.0
error_branch         8.0
error_ring           7.0
error_atom_count     2.0
error_bond_seq       1.0

# epoch=4
 
评估长度5: 
Correct: 99.00%, Predict selfies valid: 100.00%, error_type
NO                97.0
error_atom_seq     2.0
error_ring         1.0

评估长度6:
Correct: 85.00%, Predict selfies valid: 100.00%, error_type
NO                84.0
error_ring         7.0
error_atom_seq     6.0
error_branch       2.0
error_bond_seq     1.0

评估长度7:
Correct: 68.00%, Predict selfies valid: 88.00%, error_type
NO                  66.0
error_branch        13.0
error_atom_seq      11.0
error_ring           6.0
error_other          2.0
error_atom_count     2.0

# epoch=6
 
评估长度5: 
Correct: 100.00%, Predict selfies valid: 99.00%, error_type
NO                98.0
error_atom_seq     2.0

评估长度6:
Correct: 89.00%, Predict selfies valid: 98.00%, error_type
NO                  87.0
error_ring           6.0
error_atom_seq       5.0
error_atom_count     1.0
error_branch         1.0

评估长度7:
Correct: 72.00%, Predict selfies valid: 95.00%, error_type
NO                  68.0
error_atom_seq      14.0
error_ring           8.0
error_branch         7.0
error_other          1.0
error_bond_count     1.0
error_atom_count     1.0

``` 

对比基线, 没有提升, 怀疑是数据量太小

  

  

**增大branch的数据量** : 原来是将某一个原子替换成[branch1], 改为对每一个原子进行尝试, 数据量分布 (长度5/6 的数据量变为 26/92): 

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
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/raw/length_5.train.json:26
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:26
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/raw/length_6.train.json:92
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:92
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:13
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:13
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:24
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:24
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_6.train.json:40
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_6.train.json:40
``` 

  

训练结果: 

```
# epoch=1
 
评估长度5:
Correct: 96.00%, Predict selfies valid: 99.00%, error_type
NO                  95.0
error_bond_count     2.0
error_bond_seq       1.0
error_ring           1.0
error_other          1.0

评估长度6:
Correct: 76.00%, Predict selfies valid: 94.00%, error_type
NO                  73.0
error_atom_seq      14.0
error_ring           5.0
error_bond_count     3.0
error_branch         3.0
error_bond_seq       1.0
error_other          1.0
 
评估长度7:
Correct: 67.00%, Predict selfies valid: 90.00%, error_type
NO                  66.0
error_atom_seq      13.0
error_ring           9.0
error_bond_count     4.0
error_bond_seq       3.0
error_branch         3.0
error_atom_count     2.0

# epoch=2
 
评估长度5:
Correct: 98.00%, Predict selfies valid: 100.00%, error_type
NO            98.0
error_ring     2.0

评估长度6:
Correct: 85.00%, Predict selfies valid: 95.00%, error_type
NO                  81.0
error_atom_seq      11.0
error_ring           5.0
error_bond_seq       1.0
error_other          1.0
error_atom_count     1.0

评估长度7:
Correct: 71.00%, Predict selfies valid: 96.00%, error_type
NO                  70.0
error_atom_seq      11.0
error_ring           7.0
error_branch         5.0
error_atom_count     3.0
error_bond_count     3.0
error_other          1.0
 
# epoch=3
 
评估长度5:
Correct: 99.00%, Predict selfies valid: 99.00%, error_type
NO                  97.0
error_bond_seq       1.0
error_bond_count     1.0
error_ring           1.0

评估长度6:
Correct: 86.00%, Predict selfies valid: 95.00%, error_type
NO                  81.0
error_atom_seq      12.0
error_ring           4.0
error_branch         1.0
error_atom_count     1.0
error_bond_count     1.0

评估长度7:
Correct: 72.00%, Predict selfies valid: 96.00%, error_type
NO                  70.0
error_ring          11.0
error_atom_seq       8.0
error_bond_seq       4.0
error_atom_count     3.0
error_bond_count     2.0
error_branch         2.0

# epoch=4
 
评估长度5:
Correct: 99.00%, Predict selfies valid: 100.00%, error_type
NO                  99.0
error_bond_count     1.0

评估长度6:
Correct: 91.00%, Predict selfies valid: 90.00%, error_type
NO                  85.0
error_atom_seq       8.0
error_ring           4.0
error_bond_count     1.0
error_atom_count     1.0
                     1.0

评估长度7:
Correct: 62.00%, Predict selfies valid: 93.00%, error_type
NO                  59.0
error_atom_count    16.0
error_ring           8.0
error_atom_seq       7.0
error_branch         4.0
error_bond_count     3.0
error_bond_seq       2.0
error_other          1.0
 
 
``` 

对比"生成原子交换的增强数据"的结果, 每一个epoch中, 对长度7的评估中, error_branch均下降, 也就是说训练数据是有效的, 但准确率并没有提升, 错误发生了转移 (转移没有固定方向, error_ring/error_bond_count/error_bond_seq等都有)

  

epoch=3, 长度7的评估日志: [9.3.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/9.3.log), 分析日志: [9.3.analyzed.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/9.3.analyzed.log), 其中: 

  - 6个case, 与"生成错误的branch"有关, 
  - 7个case, 与"[Branch][Ring]相关"

模型会在涉及[Ring][Ring] 和 [Branch][Ring] 时, 错误地使用[branch], 或者完全不出现ring结构. 好像是 branch 和 branch/ring 和 ring/ring的区分没有那么明确 (也可能是原子7以后, branch/ring涉及的分子比例变高)

  

  

**再次增大branch的数据量** : 原来是对每一个原子进行尝试替换成[branch1], 调整为替换后再找到合适的键, 成功率变高, 数据量分布 (长度5/6 的数据量变为 79/255): 

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
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/raw/length_5.train.json:64
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/enhanced_by_image/length_5.train.json:64
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/raw/length_6.train.json:204
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies_from_chain_selfies/enhanced_by_image/length_6.train.json:204
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:13
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:13
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:24
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:24
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_6.train.json:40
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_6.train.json:40

``` 

训练效果: 

```
# epoch=1
 
评估长度5:
Correct: 92.00%, Predict selfies valid: 98.00%, error_type
NO                91.0
error_atom_seq     5.0
error_branch       2.0
error_ring         1.0
error_other        1.0

评估长度6:
Correct: 88.00%, Predict selfies valid: 95.00%, error_type
NO                  85.0
error_atom_seq       7.0
error_ring           5.0
error_atom_count     1.0
error_bond_count     1.0
error_bond_seq       1.0

 
评估长度7:
Correct: 63.00%, Predict selfies valid: 89.00%, error_type
NO                  63.0
error_atom_seq      14.0
error_ring           7.0
error_bond_seq       6.0
error_branch         4.0
error_bond_count     3.0
error_atom_count     3.0

# epoch=2
 
评估长度5:
Correct: 99.00%, Predict selfies valid: 99.00%, error_type
NO                  96.0
error_branch         2.0
error_atom_seq       1.0
error_atom_count     1.0

评估长度6:
Correct: 90.00%, Predict selfies valid: 98.00%, error_type
NO                  85.0
error_atom_seq       6.0
error_ring           5.0
error_bond_seq       2.0
error_atom_count     1.0
error_branch         1.0

评估长度7:
Correct: 74.00%, Predict selfies valid: 95.00%, error_type
NO                  70.0
error_atom_seq      12.0
error_ring           7.0
error_branch         4.0
error_bond_seq       3.0
error_atom_count     2.0
error_bond_count     2.0

 
# epoch=3
 
评估长度5:
Correct: 98.00%, Predict selfies valid: 99.00%, error_type
NO                  92.0
error_branch         4.0
error_atom_seq       2.0
error_bond_count     2.0

评估长度6:
Correct: 92.00%, Predict selfies valid: 97.00%, error_type
NO                  86.0
error_ring           5.0
error_atom_seq       4.0
error_bond_count     2.0
error_branch         2.0
error_atom_count     1.0

评估长度7:
Correct: 75.00%, Predict selfies valid: 94.00%, error_type
NO                  72.0
error_atom_seq      15.0
error_ring           6.0
error_branch         3.0
error_atom_count     2.0
error_bond_count     1.0
error_bond_seq       1.0

``` 

评估效果: 长度6的准确率有较大提升 (数据量增多), 长度7的准确率 略好于之前

对epoch=2评估长度7的结果进行分析: [6.analyzed.4.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/6.analyzed.4.log)

  - 其中有7个case是 branch和ring混淆 (branch的训练数据过多, 导致将ring错判成branch结构)

对epoch=3评估长度7的结果进行分析: [9.analyzed.4.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/9.analyzed.4.log)

  - 其中有4个case是 branch和ring混淆 (branch的训练数据过多, 导致将ring错判成branch结构)
  - 其中有3个case是ring相关 (应当识别出ring结构, 但没有识别出来) (ring的数据多样性过低)
  - 其中有9个case, 是通过原子交换可修正的

下一步想法: 

  - 整理目前的情况: branch的训练数据比较多, ring的训练数据比较少, 两者之间的关系是割裂的
  - 考虑将branch/ring的训练数据进行重新生成, 使用课程学习 (先学习基础结构, 然后学习branch结构, ring结构, 最后学习联合结构)

  

# 尝试理解branch/ring的解析原理

论文说明: 

```
这篇文章描述了 SELFIES（自引用嵌入式字符串）这种分子表征方法，以及它的 Python 实现库 `selfies`。文章详细解释了 SELFIES 语法和如何将 SELFIES 字符串解析成 SMILES 字符串。其中，[Branch] 符号用于表示分支结构。

以下是一个解析包含 [Branch] 符号的 SELFIES 字符串的例子，并结合文章内容逐步解释：

**SELFIES 字符串:** `[C][=O][Branch1][C][=O]`  代表乙酰基 (CH3CO)

**解析过程:**

1. **初始化:** 解析从起始符号 S 开始。

2. **处理原子符号:**
    * `[C]` 与 S 匹配，应用规则 S → CX4，得到字符串 `CX4`。
    * `[=O]` 与 X4 匹配，应用规则 X4 → =OX2，得到字符串 `C=OX2`。

3. **处理分支符号:**
    * 遇到 `[Branch1]`，由于当前非终止符是 X2 (i ≥ 2)，触发分支解析。
    * `[Branch1]` 中的 `1` 表示需要从 SELFIES 字符串中读取接下来的一个符号作为分支的长度索引 (用十六进制解释)。
    * 读取下一个符号 `[C]`，对应索引值为 0 (参见文章中的表3)。
    * 计算分支长度 N = 1 + 16⁰ * 0 = 1。
    * 从 SELFIES 字符串中取出接下来的 N=1 个符号，构成子字符串 `[=O]`。
    * **递归解析分支:**  以 X2 为起始符号（由 `[Branch1]` 前的 `=O` 决定， d(b) = min(2, 2, d(=)) = 2）解析子字符串 `[=O]`。
        * `[=O]` 与 X2 匹配，应用规则 X2 → =O，得到字符串 `=O`。
    * 将递归解析的结果 `=O` 用括号括起来，得到 `(=O)`。
    * 将 `(=O)` 代入原字符串，同时 X2 变为 X1 (X2 → (分支)X2-min(2-1, d(=)) = X1)，得到 `C=O(=O)X1`。

4. **处理剩余原子符号:**
    * `[C]` 与 X1 匹配，应用规则 X1 → CX3，得到 `C=O(=O)CX3`。

最终得到的 SMILES 字符串为 `C=O(=O)C`，这正是乙酰基的 SMILES 表示。

**总结:**  解析 [Branch] 符号的关键在于递归地解析分支子字符串，并根据分支符号和前一个原子符号确定分支的连接方式和剩余价态。 这个例子展示了 SELFIES 如何通过其严格的语法和自引用机制，确保生成的 SMILES 字符串的有效性。
 
 
------
 
在 SELFIES 的语法中，X4、X2 这些符号被称为**非终止符 (nonterminal symbols)**，它们在 SELFIES 字符串到 SMILES 字符串的**推导过程**中起到关键作用，代表了**原子当前剩余的价态 (remaining valence)**。

更具体地说，`Xn` 表示当前正在处理的原子*最多*还可以形成 `n` 个键。

* **初始状态 S:**  推导过程从非终止符 `S` 开始，它表示可以连接任何原子。

* **应用原子符号:** 当遇到一个原子符号 `[ba]` 时，会根据原子类型 `type(a)` 的价态 `n(type(a))` 将 `S` 或 `Xn` 替换成 `a~Xn(type(a))`。 其中 `a~` 是 `a` 转换成 SMILES 语法的表示。例如，`[C]` 会将 `S` 替换成 `CX4`，因为碳的默认价态是 4。

* **应用键符号和更新价态:**  当后续添加原子时，键符号 `b` 和前一个原子的剩余价态 `Xn` 共同决定了新键的阶数以及如何更新剩余价态。例如，如果当前字符串是 `CX4`，下一个原子符号是 `[=O]`，则会应用规则 `X4 → =OX2`。 这里 `=` 表示双键，`X2` 表示碳原子在形成双键后还剩下 2 个价态可以形成其他键。

* **分支和环:**  在处理分支符号 `[Branch]` 和环符号 `[Ring]` 时，非终止符 `Xn` 也用于确定分支和环的连接点以及如何更新剩余价态。

因此，`X4`、`X2` 等非终止符在 SELFIES 的推导过程中扮演着跟踪原子价态的角色，确保生成的分子结构符合化学规则。它们本身不会出现在最终的 SMILES 字符串中，只是推导过程中的中间状态。
``` 

````
这篇论文中, [O][C][=Branch2][C][Ring1][=O][F][=C] 被解析成了 'OC(=O)C', 我不太理解
 
https://poe.com/s/9rZSyySbXgRTfg8Bjq9R
 
---
 
您好，我来帮助您理解为什么在论文中，SELFIES 字符串 `[O][C][=Branch2][C][Ring1][=O][F][=C]` 被解析成了 SMILES 字符串 `OC(=O)C`。

我们可以按照论文中描述的 SELFIES 解码过程，逐步解析这个 SELFIES 字符串。

**给定的 SELFIES 字符串：**

```
[O][C][=Branch2][C][Ring1][=O][F][=C]
```

**步骤如下：**

---

### **步骤 1：处理 `[O]`**

- **当前状态：** 初始状态 `S`
- **SELFIES 符号：** `[O]`
- **动作：**
  - `[O]` 是一个原子符号，表示氧原子 `O`。
  - 根据氧原子的价数（默认是 2），应用规则：
    ```
    S → 'O' X2
    ```
- **结果：**
  - **当前字符串：** `'O' X2`
  - **解释：** `'O'` 是氧原子，`X2` 表示氧原子还可以形成 2 个键。

---

### **步骤 2：处理 `[C]`**

- **当前状态：** `'O' X2`
- **SELFIES 符号：** `[C]`
- **动作：**
  - `[C]` 是一个原子符号，表示碳原子 `C`。
  - 碳原子的价数是 4。
  - 连接键为默认单键，记作 `''`，键阶为 1。
  - 计算：
    ```
    d0 = min(价数, 当前非终结符的指数, 键阶) = min(4, 2, 1) = 1
    ```
  - 应用规则：
    ```
    X2 → '' 'C' X(4 - 1) = 'C' X3
    ```
- **结果：**
  - **当前字符串：** `'O' 'C' X3`
  - **解释：** `'C'` 是碳原子，`X3` 表示该碳原子还可以形成 3 个键。

---

### **步骤 3：处理 `[=Branch2]`**

- **当前状态：** `'O' 'C' X3`
- **SELFIES 符号：** `[=Branch2]`
- **动作：**
  - `[=Branch2]` 是一个分支符号，表示将要添加一个分支，键为双键（`=`），`ℓ = 2`。
  - 当前非终结符为 `X3`，指数 `i = 3`。
  - 计算：
    ```
    d(b) = 2 (因为是双键 '=')
    d0 = min(i - 1, d(b)) = min(2, 2) = 2
    ```
  - 应用规则：
    ```
    X3 → r X(3 - 2) = r X1
    ```
    其中，`r` 是通过递归解析得到的分支字符串。

- **确定分支的内容：**
  - 我们需要从 SELFIES 符号序列中取出接下来的 `ℓ` 个符号来确定分支索引：
    - **符号 4**：`[C]` 对应索引 0
    - **符号 5**：`[Ring1]` 对应索引 1
  - 计算分支索引：
    ```
    N = 1 + 16^(ℓ - 1) * c1 + 16^(ℓ - 2) * c2
      = 1 + 16^(2 - 1) * 0 + 16^(2 - 2) * 1
      = 1 + 0 + 1 = 2
    ```
  - 然后从剩余的 SELFIES 符号中取出接下来的 `N` 个符号作为分支的 SELFIES 子串：
    - **符号 6**：`[=O]`
    - **符号 7**：`[F]`
  - 分支的 SELFIES 子串为：`[=O][F]`

---

### **步骤 4：递归解析分支 `[=O][F]`**

- **起始非终结符：** `X_d0 = X2`（因为 `d0 = 2`）
- **解析 `[=O]`：**
  - `[=O]` 是一个原子符号，表示带双键的氧原子。
  - 键阶 `d(b) = 2`。
  - 计算：
    ```
    d0' = min(价数, 当前非终结符的指数, 键阶) = min(2, 2, 2) = 2
    ```
  - 应用规则：
    ```
    X2 → '=' 'O' X(2 - 2) = '=' 'O' X0
    ```
- **解析 `[F]`：**
  - 当前非终结符为 `X0`，表示无法再连接更多原子，所以解析结束。
- **结果：**
  - 分支字符串为：`'=O'`
  - 包装成 SMILES 分支表示：`'(=O)'`

---

### **步骤 5：回到主解析**

- **当前状态：** `'O' 'C' X3`
- **应用分支解析结果：**
  - 用分支字符串替换之前的非终结符：
    ```
    X3 → '(=O)' X1
    ```
- **结果：**
  - **当前字符串：** `'O' 'C' '(=O)' X1`

---

### **步骤 6：处理 `[=C]`**

- **当前状态：** `'O' 'C' '(=O)' X1`
- **SELFIES 符号：** `[=C]`
- **动作：**
  - `[=C]` 是一个原子符号，表示带双键的碳原子。
  - 键阶 `d(b) = 2`（因为是双键 '='）。
  - 计算：
    ```
    d0 = min(价数, 当前非终结符的指数, 键阶) = min(4, 1, 2) = 1
    ```
  - 应用规则：
    ```
    X1 → '' 'C' X(4 - 1) = 'C' X3
    ```
- **结果：**
  - **当前字符串：** `'O' 'C' '(=O)' 'C' X3`

---

### **步骤 7：结束解析**

- **当前状态：** `'O' 'C' '(=O)' 'C' X3`
- **动作：**
  - 由于 SELFIES 符号序列已处理完，且当前非终结符为 `X3`。
  - 根据规则，当没有更多符号时，非终结符 `Xi`（`i ≥ 1`）直接替换为空串。
    ```
    Xi → ''
    ```
- **结果：**
  - **最终字符串：** `'O' 'C' '(=O)' 'C'`

---

### **最终结果**

我们得到的 SMILES 字符串是：

```
OC(=O)C
```

这就是为什么 SELFIES 字符串 `[O][C][=Branch2][C][Ring1][=O][F][=C]` 被解析成了 SMILES 字符串 `OC(=O)C`。

---

**总结：**

- SELFIES 中的分支和环符号通过复杂的规则映射到 SMILES 表示。
- 分支符号 `[=Branch2]` 引入了一个分支，该分支的内容由后续符号确定。
- 通过递归解析分支，我们得到了分支的 SMILES 表示。
- 最终，将所有部分组合起来，得到完整的 SMILES 字符串。

希望这个详细的步骤解析能帮助您理解其中的过程。如有任何疑问，请随时提问！
```` 

关键信息, [Branch1] 后面的一个原子, 是标记分支的长度, 不是实际原子意义. 其与长度的对应表如下: 

![image2025-1-21 19:10:42.png](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/image2025-1-21%2019%3A10%3A42.png)

也就是说, branch数据的生成, 需要针对不同分支长度进行. ring的数据生成, 不要包括 [branch][ring] 这种结构, 会有干扰

# 生成规范的branch训练数据

生成方法: 

  1. 将selfies表达式分为: 前主链, 操作词, 分支, 后主链. 一共四部分.
  2. 穷举: 总体长度/操作词位置/操作词, 这样就确定了四部分的长度.
  3. 将 前主链/分支/后主链 这三部分进行随机原子生成, 作为初始的selfies
  4. 对三部分中的某一部分, 进行替换, 作为数据增强
  5. 将初始selfies, 和三部分分别替换的selfies, 作为训练数据

训练数据: 增加了branch_selfies

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
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:13
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:13
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:24
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:24
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_6.train.json:40
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_6.train.json:40
``` 

训练结果: 

```
# epoch=1
 
评估长度5:
Correct: 91.00%, Predict selfies valid: 95.00%, error_type
NO                  87.0
error_ring           4.0
error_bond_count     3.0
error_atom_seq       3.0
error_branch         2.0
error_bond_seq       1.0

评估长度6: 
Correct: 82.00%, Predict selfies valid: 94.00%, error_type
NO                  80.0
error_atom_seq       5.0
error_branch         4.0
error_bond_seq       3.0
error_atom_count     3.0
error_bond_count     3.0
error_ring           1.0
error_other          1.0
 
评估长度7:
Correct: 67.00%, Predict selfies valid: 92.00%, error_type
NO                  61.0
error_atom_seq      19.0
error_atom_count     7.0
error_ring           7.0
error_branch         4.0
error_bond_seq       2.0

 
# epoch=2

评估长度5:
Correct: 95.00%, Predict selfies valid: 98.00%, error_type
NO                  94.0
error_bond_count     2.0
error_atom_seq       2.0
error_ring           2.0

评估长度6: 
Correct: 89.00%, Predict selfies valid: 98.00%, error_type
NO                  85.0
error_atom_seq       7.0
error_ring           4.0
error_branch         1.0
error_bond_count     1.0
error_atom_count     1.0
error_other          1.0

评估长度7:
Correct: 75.00%, Predict selfies valid: 92.00%, error_type
NO                  72.0
error_atom_seq      11.0
error_ring           9.0
error_atom_count     3.0
error_branch         3.0
error_bond_count     1.0
error_other          1.0

# epoch=3

评估长度5:
Correct: 97.00%, Predict selfies valid: 95.00%, error_type
NO                93.0
error_ring         4.0
error_atom_seq     2.0
error_branch       1.0

评估长度6: 
Correct: 91.00%, Predict selfies valid: 98.00%, error_type
NO                  89.0
error_atom_count     4.0
error_atom_seq       3.0
error_ring           2.0
error_branch         2.0

评估长度7:
Correct: 72.00%, Predict selfies valid: 90.00%, error_type
NO                  66.0
error_atom_seq      10.0
error_ring           9.0
error_atom_count     7.0
error_branch         4.0
error_bond_seq       2.0
error_other          1.0
error_bond_count     1.0

# epoch=4

评估长度5:
Correct: 98.00%, Predict selfies valid: 100.00%, error_type
NO                  94.0
error_branch         2.0
error_atom_seq       2.0
error_bond_count     1.0
error_ring           1.0

评估长度6: 
Correct: 94.00%, Predict selfies valid: 99.00%, error_type
NO                  88.0
error_atom_seq       5.0
error_ring           4.0
error_bond_count     2.0
error_atom_count     1.0

评估长度7:
Correct: 73.00%, Predict selfies valid: 91.00%, error_type
NO                  71.0
error_atom_seq       9.0
error_ring           8.0
error_branch         5.0
error_atom_count     4.0
error_other          2.0
error_bond_count     1.0

# epoch=6

评估长度5:
Correct: 98.00%, Predict selfies valid: 99.00%, error_type
NO                  95.0
error_ring           2.0
error_atom_seq       2.0
error_bond_count     1.0

评估长度6: 
Correct: 91.00%, Predict selfies valid: 97.00%, error_type
NO                87.0
error_atom_seq     7.0
error_ring         4.0
error_bond_seq     1.0
                   1.0	

评估长度7:
Correct: 70.00%, Predict selfies valid: 89.00%, error_type
NO                  67.0
error_ring          11.0
error_atom_seq      11.0
error_branch         6.0
error_bond_seq       2.0
error_bond_count     1.0
error_atom_count     1.0
                     1.0
``` 

对SOTA (epoch=2, 长度7) 的评估结果: [6.analyzed.5.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/6.analyzed.5.log)

  - 其中8个 "[Ring]识别错误"
  - 其中7个 "[Branch]识别错误"
    - 其中有6个case, 是应当识别出branch结构, 但没有识别出来
      - 其中有一个"[Branch1][Ring2]" 出现在位置2 (在训练数据的长度5/6的数据中不存在)
      - 其中有一个"[Branch1][Ring1]" 出现在位置3 (在训练数据的长度5/6的数据中不存在)

对 (epoch=3, 长度7) 的评估结果: [9.analyzed.5.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/9.analyzed.5.log)

  - 其中8个 "[Ring]识别错误"
  - 其中5个"[Branch]识别错误"
    - 包括一个"[Branch1][Ring2]" 出现在位置2 (在训练数据的长度5/6的数据中不存在)

结论: 使用规范数据集后, 整体效果并没有明显的变化

# 在提示词中增加对[branch]的理解

commit: 276777285a4b3a105f20839823d1562c04948f20

在提示词中增加了以下描述: 

```
### How to understand [Branch1]
In **SELFIES**, `[Branch1]` introduces a branch, and the symbols following it determine how many atoms are included in the branch based on a predefined index table. Each symbol corresponds to a specific branch length:

- **`[Branch1][C]`**: The branch includes **1 atom**.  
- **`[Branch1][Ring1]`**: The branch includes **2 atoms**.  
- **`[Branch1][Branch1]`**: The branch includes **4 atoms**.  
- **`[Branch1][#Branch1]`**: The branch includes **6 atoms**.  

**Example:**

SELFIES string `[C][C][Branch1][Ring1][O][C]`:

1. `[C][C]`: Two carbon atoms in a chain.  
2. `[Branch1][Ring1]`: The branch includes **2 atoms** (`[O][C]`) attached to the second carbon.  

The resulting molecule is a linear chain with a branch containing oxygen and carbon.

``` 

从训练评估效果上, 并没有提升

训练日志: [run_tasks.log](/assets/01KJBZQHK4TKYKDV1XKS00J7CM/run_tasks.log)

# 在规范数据集上, 增大branch训练数据的数据量

注意: 长度5的数据量无法增加, 长度6的数据量可以翻倍 (增加原子替换的"增强数据")

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
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/raw/length_6.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/branch_selfies/enhanced_by_image/length_6.train.json:160
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_5.train.json:13
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_5.train.json:13
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/raw/length_6.train.json:24
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count/ring_selfies/enhanced_by_image/length_6.train.json:24
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_5.train.json:20
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/raw/length_6.train.json:40
/opt/huangyan/mol-ai/datasets/cot_with_CNO_count.enhance_by_randomize_selfies/ring_selfies/enhanced_by_image/length_6.train.json:40
``` 

训练效果: 

```
# epoch=1
评估长度5: 
Correct: 96.00%, Predict selfies valid: 95.00%, error_type
NO                  91.0
error_atom_seq       6.0
error_ring           2.0
error_atom_count     1.0

评估长度6: 
Correct: 89.00%, Predict selfies valid: 93.00%, error_type
NO                  87.0
error_ring           5.0
error_atom_seq       3.0
error_branch         2.0
error_bond_count     2.0
error_bond_seq       1.0

评估长度7: 
Correct: 65.00%, Predict selfies valid: 91.00%, error_type
NO                  62.0
error_atom_seq      13.0
error_ring           9.0
error_branch         7.0
error_atom_count     5.0
error_bond_seq       2.0
error_bond_count     2.0

# epoch=2

评估长度5: 
Correct: 99.00%, Predict selfies valid: 99.00%, error_type
NO                95.0
error_atom_seq     3.0
error_ring         2.0

评估长度6: 
Correct: 94.00%, Predict selfies valid: 97.00%, error_type
NO                  90.0
error_atom_seq       4.0
error_ring           3.0
error_bond_count     1.0
error_branch         1.0
error_bond_seq       1.0

评估长度7: 
Correct: 67.00%, Predict selfies valid: 87.00%, error_type
NO                  65.0
error_atom_seq      11.0
error_branch         8.0
error_ring           7.0
error_bond_count     3.0
error_atom_count     3.0
error_bond_seq       2.0
error_other          1.0

# epoch=3

评估长度5: 
Correct: 99.00%, Predict selfies valid: 99.00%, error_type
NO                  96.0
error_atom_seq       2.0
error_bond_count     1.0
error_ring           1.0

评估长度6: 
Correct: 93.00%, Predict selfies valid: 96.00%, error_type
NO                  89.0
error_atom_seq       4.0
error_ring           2.0
error_bond_count     2.0
error_other          1.0
error_branch         1.0
error_bond_seq       1.0

评估长度7: 
Correct: 69.00%, Predict selfies valid: 90.00%, error_type
NO                  65.0
error_atom_seq      14.0
error_branch         7.0
error_ring           7.0
error_bond_count     3.0
error_atom_count     3.0
error_other          1.0

# epoch=4

评估长度5: 
Correct: 99.00%, Predict selfies valid: 100.00%, error_type
NO                  97.0
error_atom_seq       2.0
error_bond_count     1.0

评估长度6: 
Correct: 93.00%, Predict selfies valid: 97.00%, error_type
NO                  89.0
error_atom_seq       4.0
error_ring           3.0
error_branch         2.0
error_bond_count     2.0

评估长度7: 
Correct: 72.00%, Predict selfies valid: 87.00%, error_type
NO                  68.0
error_ring          10.0
error_atom_seq       9.0
error_atom_count     4.0
error_branch         4.0
error_bond_count     3.0
error_other          1.0
error_bond_seq       1.0

``` 

分析评估日志, 也没有太大变化

结论: 增加[branch]规范数据的多样性 (增加原子替换, 在同样的结构下生成更多种类的原子组合), 并不能改善效果. SOTA持续在72%附近

# 下一步

先放弃对[branch]的优化, 目前没有更好的策略.

先增加[ring]的规范数据, 看看有没有其他想法.
