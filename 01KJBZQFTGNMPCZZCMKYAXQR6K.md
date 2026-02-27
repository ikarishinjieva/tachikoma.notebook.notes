---
title: 20250115 - 使用ms-swift对Qwen2-VL进行微调 [19] - 调优长度7的效果
confluence_page_id: 3343876
created_at: 2025-01-16T15:45:39+00:00
updated_at: 2025-01-18T04:55:37+00:00
---

# 前继

在之前的实验中, 最好的训练方式如下: 

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 从长度5开始有ring的数据增强, 并为了长度6的ring数据增加单独的一轮单一数据

  1. 开始训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
  2. 继续训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     3. 识别任务, 包括ring的长度5的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度5的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)
  3. 继续训练, 单一数据: epoch+1  

     1. 识别任务, 包括ring的长度6的训练数据, 带图形变换的数据增强
     2. 识别任务, 包括ring的长度6的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)

在长度6达到了83%的正确率, 但在长度7的正确率不高: 

Correct: 43.00%, Predict selfies valid: 85.00%, error_type  
NO 40.0  
error_atom_seq 28.0  
error_ring 9.0  
error_atom_count 7.0  
error_branch 6.0  
error_bond_seq 5.0  
error_bond_count 4.0  
error_other 1.0

# 增加少量长度6的数据 (基线)

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 从长度5开始有ring的数据增强, 增加少量长度6的数据

  1. 开始训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
  2. 继续训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     3. 识别任务, 包括ring的长度5的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度5的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)
  3. 继续训练, 混合数据: epoch+1  

     1. 识别任务, 带图形变换的数据增强, 长度1-6数据, 每种最多100个
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-6数据, 每种最多100个
     3. 识别任务, 包括ring的长度6的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度6的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)  
  

```
# epoch=1

评估长度6:
Correct: 62.00%, Predict selfies valid: 90.00%, error_type
NO 61.0
error_atom_seq 19.0
error_bond_count 11.0
error_branch 3.0
error_bond_seq 2.0
error_ring 2.0
error_other 1.0
error_atom_count 1.0

评估长度7: 
Correct: 48.00%, Predict selfies valid: 80.00%, error_type
NO 47.0
error_atom_seq 25.0
error_ring 9.0
error_branch 7.0
error_bond_count 7.0
error_bond_seq 3.0
error_other 1.0
error_atom_count 1.0

 
# epoch=2

评估长度6
Correct: 81.00%, Predict selfies valid: 93.00%, error_type
NO 77.0
error_atom_seq 12.0
error_ring 3.0
error_bond_seq 3.0
error_branch 2.0
error_bond_count 2.0
error_other 1.0

评估长度7
Correct: 60.00%, Predict selfies valid: 87.00%, error_type
NO 55.0
error_atom_seq 24.0
error_branch 7.0
error_ring 6.0
error_bond_count 5.0
error_bond_seq 2.0
error_atom_count 1.0

 
# epoch=3
 
评估长度6
Correct: 89.00%, Predict selfies valid: 99.00%, error_type
NO 84.0
error_atom_seq 7.0
error_ring 4.0
error_branch 2.0
error_bond_seq 2.0
error_other 1.0

评估长度7
Correct: 50.00%, Predict selfies valid: 84.00%, error_type
NO 45.0
error_atom_count 28.0
error_ring 8.0
error_atom_seq 8.0
error_branch 7.0
error_bond_count 2.0
error_bond_seq 2.0
 
# epoch=4:
评估长度6:Correct: 86.00%, Predict selfies valid: 98.00%, error_type
NO 83.0
error_atom_seq 6.0
error_ring 4.0
error_bond_seq 3.0
error_branch 1.0
error_other 1.0
error_bond_count 1.0
error_atom_count 1.0

评估长度7
Correct: 65.00%, Predict selfies valid: 89.00%, error_type
NO 60.0
error_atom_seq 16.0
error_ring 7.0
error_atom_count 7.0
error_branch 6.0
error_bond_count 2.0
error_other 1.0
error_bond_seq 1.0
``` 

  

  - 评估长度7时, 准确率下降, 大部分分子被预测成长度8
  - 长度7的评估日志(epoch=4): [evaluate_8.log](/assets/01KJBZQFTGNMPCZZCMKYAXQR6K/evaluate_8.log) 

  

# 尝试简化步骤: 使用长度5+长度6(部分)

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 从长度5开始有ring的数据增强, 增加少量长度6的数据

  1. ~~开始训练, 混合数据: epoch+1~~
     1. ~~识别任务, 带图形变换的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)~~
     2. ~~识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)~~
  2. 继续训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     3. 识别任务, 包括ring的长度5的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度5的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)
  3. 继续训练, 混合数据: epoch+1  

     1. 识别任务, 带图形变换的数据增强, 长度1-6数据, 每种最多100个
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-6数据, 每种最多100个
     3. 识别任务, 包括ring的长度6的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度6的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)

  

```
# epoch=1

评估长度5:
Correct: 83.00%, Predict selfies valid: 89.00%, error_type
NO                  83.0
error_atom_seq       7.0
error_bond_count     3.0
error_bond_seq       2.0
error_atom_count     2.0
error_ring           2.0
error_branch         1.0

评估长度6:
Correct: 71.00%, Predict selfies valid: 93.00%, error_type
NO                  71.0
error_atom_seq      12.0
error_bond_seq       5.0
error_bond_count     4.0
error_branch         3.0
error_ring           3.0
error_atom_count     2.0

评估长度7: 
Correct: 47.00%, Predict selfies valid: 81.00%, error_type
NO                  43.0
error_atom_seq      18.0
error_atom_count    15.0
error_ring           8.0
error_branch         7.0
error_bond_count     6.0
error_bond_seq       2.0
                     1.0

 
# epoch=2

评估长度5
Correct: 90.00%, Predict selfies valid: 95.00%, error_type
NO                  88.0
error_atom_seq       3.0
error_atom_count     2.0
error_bond_seq       2.0
error_bond_count     2.0
error_branch         2.0
error_ring           1.0
评估长度6
Correct: 76.00%, Predict selfies valid: 91.00%, error_type
NO                  72.0
error_atom_seq      15.0
error_bond_count     3.0
error_bond_seq       3.0
error_branch         3.0
error_ring           2.0
error_atom_count     1.0
                     1.0

评估长度7
Correct: 35.00%, Predict selfies valid: 85.00%, error_type
error_atom_count    46.0
NO                  33.0
error_ring           8.0
error_branch         8.0
error_atom_seq       2.0
                     2.0
error_bond_count     1.0

 
# epoch=3

评估长度5
Correct: 88.00%, Predict selfies valid: 91.00%, error_type
NO                  85.0
error_atom_seq       4.0
error_bond_count     3.0
error_atom_count     3.0
error_ring           2.0
error_branch         1.0
error_other          1.0
error_bond_seq       1.0

评估长度6
Correct: 86.00%, Predict selfies valid: 95.00%, error_type
NO                  81.0
error_atom_seq      10.0
error_bond_count     4.0
error_branch         3.0
error_ring           2.0

评估长度7
Correct: 59.00%, Predict selfies valid: 82.00%, error_type
NO                  54.0
error_atom_seq      19.0
error_ring           8.0
error_branch         7.0
error_bond_count     6.0
error_atom_count     3.0
error_other          2.0
error_bond_seq       1.0

# epoch=4

评估长度5
Correct: 95.00%, Predict selfies valid: 96.00%, error_type
NO                  92.0
error_branch         3.0
error_bond_count     2.0
error_atom_seq       2.0
error_bond_seq       1.0

评估长度6
Correct: 79.00%, Predict selfies valid: 96.00%, error_type
NO                77.0
error_atom_seq    12.0
error_ring         5.0
error_bond_seq     3.0
error_branch       3.0

评估长度7
Correct: 53.00%, Predict selfies valid: 89.00%, error_type
NO                  50.0
error_atom_seq      27.0
error_ring           7.0
error_branch         6.0
error_atom_count     5.0
error_bond_seq       4.0
error_other          1.0

``` 

  - 与基线相比:
    - 都经历了一个中间阶段, 基线在(epoch=3), 本次训练在(epoch=2), 对于长度7出现了atom_count错误率上升的阶段
    - 本次训练的SOTA出现在epoch=3 (59%), 基线出现在epoch=2 (60%) / epoch=4 (65%)  

  

# 尝试简化步骤: 使用长度5+长度6(所有数据)

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 从长度5开始有ring的数据增强, 增加少量长度6的数据

  1. ~~开始训练, 混合数据: epoch+1~~
     1. ~~识别任务, 带图形变换的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)~~
     2. ~~识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)~~
  2. 继续训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     3. 识别任务, 包括ring的长度5的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度5的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)
  3. 继续训练, 混合数据: epoch+1  

     1. 识别任务, 带图形变换的数据增强, 长度1-5数据 (最多取100个), 长度6数据 (取全量, 198个)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-5数据 (最多取100个), 长度6数据 (取全量, 198个)
     3. 识别任务, 包括ring的长度6的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度6的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)

  

```
# epoch=1

评估长度5:
Correct: 87.00%, Predict selfies valid: 92.00%, error_type
NO                  84.0
error_bond_count     7.0
error_atom_seq       3.0
error_branch         2.0
                     1.0
error_other          1.0
error_bond_seq       1.0
error_ring           1.0

评估长度6:
Correct: 74.00%, Predict selfies valid: 91.00%, error_type
NO                  72.0
error_atom_seq      11.0
error_ring           5.0
error_bond_seq       4.0
error_bond_count     3.0
error_branch         2.0
                     2.0
error_atom_count     1.0

评估长度7: 
Correct: 55.00%, Predict selfies valid: 86.00%, error_type
NO                  51.0
error_atom_seq      21.0
error_ring          10.0
error_bond_count     6.0
error_branch         6.0
                     2.0
error_bond_seq       2.0
error_other          1.0
error_atom_count     1.0

 
# epoch=2

评估长度5
Correct: 89.00%, Predict selfies valid: 89.00%, error_type
NO                  85.0
error_atom_seq       6.0
error_atom_count     2.0
error_ring           2.0
error_bond_count     2.0
error_bond_seq       2.0
error_branch         1.0

评估长度6
Correct: 83.00%, Predict selfies valid: 94.00%, error_type
NO                  81.0
error_atom_seq       8.0
error_ring           3.0
error_bond_seq       2.0
error_branch         2.0
error_bond_count     2.0
error_other          1.0
error_atom_count     1.0

评估长度7
Correct: 49.00%, Predict selfies valid: 80.00%, error_type
NO                  45.0
error_atom_count    25.0
error_atom_seq      10.0
error_ring           9.0
error_branch         4.0
error_bond_count     4.0
error_other          1.0
                     1.0
error_bond_seq       1.0

 
# epoch=3

评估长度5
Correct: 92.00%, Predict selfies valid: 94.00%, error_type
NO                  91.0
error_atom_seq       3.0
error_branch         2.0
error_bond_count     2.0
error_bond_seq       1.0
error_ring           1.0

评估长度6
Correct: 86.00%, Predict selfies valid: 97.00%, error_type
NO                  84.0
error_atom_seq       8.0
error_ring           4.0
error_bond_seq       2.0
error_branch         1.0
error_bond_count     1.0

评估长度7
Correct: 65.00%, Predict selfies valid: 92.00%, error_type
NO                  61.0
error_atom_seq      13.0
error_branch         8.0
error_ring           7.0
error_atom_count     5.0
error_bond_seq       3.0
error_bond_count     2.0
                     1.0

# epoch=4

评估长度5
Correct: 92.00%, Predict selfies valid: 91.00%, error_type
NO                  90.0
error_branch         3.0
error_atom_seq       2.0
error_bond_count     2.0
                     1.0
error_ring           1.0
error_bond_seq       1.0

评估长度6
Correct: 86.00%, Predict selfies valid: 96.00%, error_type
NO                  82.0
error_atom_seq       8.0
error_ring           4.0
error_branch         2.0
error_bond_seq       2.0
error_atom_count     1.0
error_other          1.0

评估长度7
Correct: 62.00%, Predict selfies valid: 89.00%, error_type
NO                  60.0
error_atom_seq      15.0
error_atom_count     9.0
error_ring           7.0
error_branch         5.0
error_bond_seq       3.0
                     1.0

``` 

  

  - 与基线相比:
    - SOTA出现在epoch=3 (65%), 基线出现在epoch=4 (65%), 准确率差不多

  

# 尝试简化步骤: 使用长度4+长度6(所有数据)

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 从长度5开始有ring的数据增强, 增加少量长度6的数据

  1. 开始训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
  2. ~~继续训练, 混合数据: epoch+1~~
     1. ~~识别任务, 带图形变换的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)~~
     2. ~~识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)~~
     3. ~~识别任务, 包括ring的长度5的训练数据, 带图形变换的数据增强~~
     4. ~~识别任务, 包括ring的长度5的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)~~
  3. 继续训练, 混合数据: epoch+1  

     1. 识别任务, 带图形变换的数据增强, 长度1-5数据 (最多取100个), 长度6数据 (取全量, 198个)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-5数据 (最多取100个), 长度6数据 (取全量, 198个)
     3. 识别任务, 包括ring的长度6的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度6的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)

  

```
# epoch=1

评估长度5:
Correct: 79.00%, Predict selfies valid: 91.00%, error_type
NO                  75.0
error_atom_seq      10.0
error_atom_count     4.0
error_branch         3.0
error_bond_count     3.0
error_ring           2.0
error_bond_seq       2.0
error_other          1.0

评估长度6:
Correct: 61.00%, Predict selfies valid: 87.00%, error_type
NO                  60.0
error_atom_seq      16.0
error_bond_count     8.0
error_ring           6.0
error_bond_seq       4.0
error_branch         2.0
error_other          2.0
                     1.0
error_atom_count     1.0

评估长度7: 
Correct: 34.00%, Predict selfies valid: 85.00%, error_type
NO                  33.0
error_atom_seq      22.0
error_ring          13.0
error_bond_count    11.0
error_atom_count    10.0
error_branch         7.0
error_bond_seq       4.0

 
# epoch=2

评估长度5
Correct: 83.00%, Predict selfies valid: 90.00%, error_type
NO                  81.0
error_atom_seq       5.0
error_bond_count     4.0
error_branch         3.0
error_atom_count     2.0
error_ring           2.0
error_bond_seq       2.0
error_other          1.0

评估长度6
Correct: 76.00%, Predict selfies valid: 91.00%, error_type
NO                75.0
error_atom_seq    10.0
error_bond_seq     8.0
error_ring         4.0
error_branch       3.0

评估长度7
Correct: 46.00%, Predict selfies valid: 79.00%, error_type
NO                  40.0
error_atom_count    26.0
error_atom_seq      10.0
error_ring           9.0
error_branch         7.0
error_bond_seq       4.0
error_bond_count     3.0
                     1.0

 
# epoch=3

评估长度5
Correct: 87.00%, Predict selfies valid: 92.00%, error_type
NO                  83.0
error_atom_seq       8.0
error_bond_seq       3.0
error_branch         3.0
error_bond_count     2.0
error_atom_count     1.0

评估长度6
Correct: 76.00%, Predict selfies valid: 96.00%, error_type
NO                  76.0
error_bond_seq       8.0
error_ring           4.0
error_bond_count     3.0
error_atom_seq       3.0
error_branch         3.0
error_atom_count     2.0
error_other          1.0

评估长度7
Correct: 57.00%, Predict selfies valid: 85.00%, error_type
NO                  54.0
error_atom_seq      15.0
error_ring           9.0
error_atom_count     8.0
error_branch         5.0
error_bond_seq       5.0
error_bond_count     4.0

# epoch=4

评估长度5
Correct: 90.00%, Predict selfies valid: 91.00%, error_type
NO                  88.0
error_branch         5.0
error_atom_seq       3.0
error_atom_count     1.0
error_bond_seq       1.0
error_bond_count     1.0
error_ring           1.0

评估长度6
Correct: 79.00%, Predict selfies valid: 89.00%, error_type
NO                  77.0
error_bond_count     5.0
error_atom_seq       4.0
error_ring           4.0
error_branch         3.0
error_atom_count     3.0
error_bond_seq       2.0
                     2.0

评估长度7
Correct: 56.00%, Predict selfies valid: 88.00%, error_type
NO                  53.0
error_atom_seq      15.0
error_bond_count     9.0
error_branch         8.0
error_ring           6.0
error_bond_seq       5.0
error_atom_count     4.0

``` 

  

比起上一方案, 本方案的准确率全面下降

  

# 阶段性结论

  - 长度6的数据多少, 影响了长度7的准确率
  - 可以将训练过程简化成 "使用长度5+长度6(所有数据)", 猜测是因为长度5和长度6的训练中, 本来就会包括一部分长度4的数据, 让前面长度4的步骤意义降低

  

# 下一步

考虑对现在的错误进行分析, 然后增加数据增强

在这之前, 先另起一个项目: <https://github.com/ikarishinjieva/mol-ai>

将现有的训练脚本和数据生成脚本进行整理, 否则会越来越乱
