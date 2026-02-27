---
title: 20250111 - 使用ms-swift对Qwen2-VL进行微调 [18] - 使用相似COT流程进行两种互补任务
confluence_page_id: 3343813
created_at: 2025-01-11T08:53:10+00:00
updated_at: 2025-01-14T02:15:49+00:00
---

# 前继总结

对预测的错误进行了分类, 目前面对的是atom_seq这种错误

本想通过使用相同的COT流程, 加上 "对错误表达式进行修订"的训练数据, 降低atom_seq的错误率, 但起到了相反的想过

# 基线

使用相似COT流程进行两种任务: 识别任务和修正任务. (见下一节)

在基线中: 只训练最终任务.

评估中, 评估两种方案: 

  - 一阶段评估
  - 两阶段评估 (先识别selfies, 再进行修正)

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)

  

### epoch-1

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 93.33%, error_type

NO 84.000000

error_bond_count 10.666667

error_atom_seq 5.333333

  2. 评估长度5: 
     1. Correct: 72.00%, Predict selfies valid: 91.00%, error_type

NO 72.0

error_bond_count 6.0

error_atom_seq 5.0

error_bond_seq 5.0

error_ring 5.0

error_atom_count 4.0

error_branch 3.0

  3. 评估长度6: 
     1. Correct: 50.00%, Predict selfies valid: 82.00%, error_type

NO 49.0

error_atom_seq 23.0

error_ring 9.0

error_bond_count 7.0

error_bond_seq 6.0

error_atom_count 3.0

error_branch 2.0

error_other 1.0

### epoch-1, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 85.33%, Predict selfies valid: 98.67%, error_type

NO 84.000000

error_bond_count 9.333333

error_atom_seq 5.333333

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 66.00%, Predict selfies valid: 92.00%, error_type

NO 66.0

error_bond_count 10.0

error_atom_seq 9.0

error_ring 5.0

error_atom_count 3.0

error_branch 3.0

error_bond_seq 2.0

error_other 2.0

  3. 评估长度6: 
     1. Correct: 45.00%, Predict selfies valid: 84.00%, error_type

NO 43.0

error_atom_seq 18.0

error_bond_count 14.0

error_ring 9.0

error_bond_seq 8.0

error_atom_count 6.0

error_branch 2.0

  

### epoch-2:

  1. 评估长度4: 
     1. Correct: 81.33%, Predict selfies valid: 98.67%, error_type

NO 81.333333

error_bond_count 10.666667

error_atom_seq 5.333333

error_bond_seq 1.333333

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 73.00%, Predict selfies valid: 96.00%, error_type

NO 73.0

error_bond_count 10.0

error_atom_seq 5.0

error_ring 5.0

error_atom_count 3.0

error_branch 3.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 56.00%, Predict selfies valid: 95.00%, error_type

NO 56.0

error_atom_seq 12.0

error_bond_count 10.0

error_ring 9.0

error_atom_count 5.0

error_bond_seq 4.0

error_branch 2.0

error_other 2.0

  

### epoch-2, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 77.33%, Predict selfies valid: 96.00%, error_type

NO 77.333333

error_bond_count 9.333333

error_atom_seq 9.333333

error_bond_seq 2.666667

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 72.00%, Predict selfies valid: 94.00%, error_type

NO 72.0

error_bond_count 11.0

error_ring 5.0

error_atom_count 3.0

error_atom_seq 3.0

error_branch 3.0

error_bond_seq 2.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 54.00%, Predict selfies valid: 87.00%, error_type

NO 53.0

error_atom_seq 18.0

error_ring 9.0

error_bond_count 9.0

error_atom_count 5.0

error_bond_seq 4.0

error_branch 2.0

  

### epoch-3:

  1. 评估长度4: 
     1. Correct: 92.00%, Predict selfies valid: 97.33%, error_type

NO 92.000000

error_bond_count 4.000000

error_atom_seq 2.666667

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 77.00%, Predict selfies valid: 94.00%, error_type

NO 76.0

error_bond_count 11.0

error_atom_seq 4.0

error_ring 4.0

error_branch 3.0

error_atom_count 1.0

1.0

  3. 评估长度6: 
     1. Correct: 59.00%, Predict selfies valid: 90.00%, error_type

NO 59.0

error_atom_seq 14.0

error_ring 9.0

error_atom_count 6.0

error_bond_seq 5.0

error_bond_count 5.0

error_branch 2.0

### epoch-3, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 89.33%, Predict selfies valid: 98.67%, error_type

NO 89.333333

error_bond_count 8.000000

error_atom_seq 2.666667

  2. 评估长度5: 
     1. Correct: 82.00%, Predict selfies valid: 98.00%, error_type

NO 82.0

error_bond_count 6.0

error_ring 5.0

error_branch 3.0

error_atom_count 2.0

error_other 1.0

error_atom_seq 1.0

  3. 评估长度6: 
     1. Correct: 54.00%, Predict selfies valid: 86.00%, error_type

NO 53.0

error_atom_seq 16.0

error_ring 8.0

error_bond_count 8.0

error_atom_count 7.0

error_bond_seq 5.0

error_branch 2.0

1.0

### epoch-4:

  1. 评估长度4: 
     1. Correct: 90.67%, Predict selfies valid: 97.33%, error_type

NO 90.666667

error_atom_seq 6.666667

error_bond_count 2.666667

  2. 评估长度5: 
     1. Correct: 85.00%, Predict selfies valid: 97.00%, error_type

NO 85.0

error_ring 5.0

error_branch 3.0

error_atom_count 2.0

error_atom_seq 2.0

error_bond_seq 1.0

error_bond_count 1.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 66.00%, Predict selfies valid: 95.00%, error_type

NO 66.0

error_atom_seq 15.0

error_ring 9.0

error_atom_count 4.0

error_bond_count 3.0

error_branch 2.0

error_bond_seq 1.0

  

### epoch-4, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 88.00%, Predict selfies valid: 97.33%, error_type

NO 88.000000

error_atom_seq 9.333333

error_bond_count 2.666667

  2. 评估长度5: 
     1. Correct: 83.00%, Predict selfies valid: 97.00%, error_type

NO 83.0

error_ring 5.0

error_atom_seq 4.0

error_branch 3.0

error_bond_count 3.0

error_bond_seq 1.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 64.00%, Predict selfies valid: 92.00%, error_type

NO 64.0

error_atom_seq 13.0

error_ring 9.0

error_atom_count 5.0

error_bond_count 4.0

error_bond_seq 3.0

error_branch 2.0

  

### epoch-6:

  1. 评估长度4: 
     1. Correct: 88.00%, Predict selfies valid: 98.67%, error_type

NO 88.0

error_bond_count 8.0

error_atom_seq 4.0

  2. 评估长度5: 
     1. Correct: 83.00%, Predict selfies valid: 96.00%, error_type

NO 83.0

error_bond_count 5.0

error_ring 4.0

error_branch 3.0

error_atom_seq 2.0

error_bond_seq 2.0

1.0

  3. 评估长度6: 
     1. Correct: 72.00%, Predict selfies valid: 94.00%, error_type

NO 72.0

error_ring 9.0

error_atom_seq 9.0

error_bond_count 3.0

error_atom_count 3.0

error_bond_seq 2.0

error_branch 2.0

### epoch-6, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 88.00%, Predict selfies valid: 100.00%, error_type

NO 88.000000

error_bond_count 9.333333

error_atom_seq 2.666667

  2. 评估长度5: 
     1. Correct: 80.00%, Predict selfies valid: 98.00%, error_type

NO 80.0

error_bond_count 7.0

error_ring 5.0

error_atom_seq 4.0

error_branch 3.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 74.00%, Predict selfies valid: 95.00%, error_type

NO 74.0

error_ring 9.0

error_bond_seq 5.0

error_bond_count 5.0

error_atom_seq 4.0

error_branch 2.0

error_atom_count 1.0

  

### epoch-8:

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 100.00%, error_type

NO 86.666667

error_bond_count 12.000000

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 83.00%, Predict selfies valid: 97.00%, error_type

NO 83.0

error_bond_count 6.0

error_ring 5.0

error_branch 3.0

error_atom_seq 3.0

  3. 评估长度6: 
     1. Correct: 65.00%, Predict selfies valid: 89.00%, error_type

NO 65.0

error_atom_seq 11.0

error_ring 9.0

error_atom_count 5.0

error_bond_count 5.0

error_bond_seq 3.0

error_branch 2.0

### epoch-8, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 100.00%, error_type

NO 86.666667

error_bond_count 12.000000

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 86.00%, Predict selfies valid: 97.00%, error_type

NO 85.0

error_bond_count 7.0

error_ring 4.0

error_branch 3.0

1.0

  3. 评估长度6: 
     1. Correct: 64.00%, Predict selfies valid: 87.00%, error_type

NO 63.0

error_atom_seq 12.0

error_ring 8.0

error_bond_count 7.0

error_atom_count 4.0

error_bond_seq 3.0

error_branch 2.0

1.0

  

对基线的结论:

  - 一阶段评估和两阶段评估, 性能没有差别
  - 对长度6的识别率最高的, 是 epoch=6 (74%)
  - 注意: 根据之前经验, 在多次试验中准确率的波动率会比较大

# 使用相似COT流程进行两种任务: 识别任务和修正任务

重新回到使用相似COT流程进行两个任务, 让两个任务互补

生成脚本: [loop_selfies_and_image.generate_chain.CNO_count.correct_atom_seq.cot.ipynb](/assets/01KJBZQCSH2WSBPRYJQFMYM1C3/loop_selfies_and_image.generate_chain.CNO_count.correct_atom_seq.cot.ipynb)

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 增加了对原子位置的修正任务

  1. 开始训练, 混合数据: epoch+1
     1. 修正任务, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 识别任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
  2. 继续训练, 混合数据: epoch+1
     1. 修正任务, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)

###  epoch=1  

  1. 评估长度4: 
     1. Correct: 76.00%, Predict selfies valid: 92.00%, error_type

NO 74.666667

error_atom_count 10.666667

error_bond_count 8.000000

error_atom_seq 4.000000

error_bond_seq 1.333333

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 73.00%, Predict selfies valid: 94.00%, error_type

NO 72.0

error_atom_count 10.0

error_ring 5.0

error_bond_count 5.0

error_atom_seq 4.0

error_branch 3.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 47.00%, Predict selfies valid: 88.00%, error_type

NO 45.0

error_atom_seq 17.0

error_bond_count 13.0

error_atom_count 10.0

error_ring 9.0

error_bond_seq 4.0

error_branch 2.0

  

### epoch-1, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 74.67%, Predict selfies valid: 93.33%, error_type

NO 73.333333

error_bond_count 12.000000

error_atom_count 10.666667

error_atom_seq 2.666667

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 65.00%, Predict selfies valid: 92.00%, error_type

NO 65.0

error_atom_count 9.0

error_bond_count 6.0

error_atom_seq 6.0

error_bond_seq 6.0

error_ring 5.0

error_branch 3.0

  3. 评估长度6: 
     1. Correct: 43.00%, Predict selfies valid: 92.00%, error_type

NO 43.0

error_atom_seq 13.0

error_atom_count 11.0

error_bond_count 11.0

error_bond_seq 10.0

error_ring 9.0

error_branch 2.0

error_other 1.0

  

###  epoch=2

  1. 评估长度4: 
     1. Correct: 82.67%, Predict selfies valid: 97.33%, error_type

NO 82.666667

error_bond_count 13.333333

error_atom_seq 4.000000

  2. 评估长度5: 
     1. Correct: 70.00%, Predict selfies valid: 96.00%, error_type

NO 70.0

error_bond_count 11.0

error_ring 5.0

error_atom_seq 5.0

error_atom_count 4.0

error_branch 3.0

error_bond_seq 2.0

  3. 评估长度6: 
     1. Correct: 54.00%, Predict selfies valid: 94.00%, error_type

NO 54.0

error_bond_count 13.0

error_atom_seq 10.0

error_bond_seq 8.0

error_ring 8.0

error_atom_count 4.0

error_branch 2.0

1.0

  

### epoch-2, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 69.33%, Predict selfies valid: 97.33%, error_type

NO 69.333333

error_bond_count 24.000000

error_atom_seq 5.333333

error_bond_seq 1.333333

  2. 评估长度5: 
     1. Correct: 48.00%, Predict selfies valid: 87.00%, error_type

NO 47.0

error_bond_count 19.0

error_atom_seq 14.0

error_bond_seq 8.0

error_ring 5.0

error_atom_count 4.0

error_branch 3.0

  3. 评估长度6: 
     1. Correct: 41.00%, Predict selfies valid: 84.00%, error_type

NO 39.0

error_atom_seq 15.0

error_bond_count 15.0

error_bond_seq 13.0

error_ring 9.0

error_atom_count 6.0

error_branch 2.0

error_other 1.0

  

### epoch=3

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 96.00%, error_type

NO 86.666667

error_atom_seq 8.000000

error_bond_count 5.333333

  2. 评估长度5: 
     1. Correct: 82.00%, Predict selfies valid: 96.00%, error_type

NO 82.0

error_ring 5.0

error_atom_seq 4.0

error_bond_count 4.0

error_branch 3.0

error_atom_count 1.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 71.00%, Predict selfies valid: 93.00%, error_type

NO 71.0

error_ring 8.0

error_atom_seq 7.0

error_bond_count 6.0

error_atom_count 3.0

error_branch 2.0

error_bond_seq 2.0

1.0

  

### epoch-3, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 84.00%, Predict selfies valid: 98.67%, error_type

NO 84.000000

error_atom_seq 8.000000

error_bond_count 6.666667

error_bond_seq 1.333333

  2. 评估长度5: 
     1. Correct: 82.00%, Predict selfies valid: 98.00%, error_type

NO 82.0

error_ring 5.0

error_atom_seq 4.0

error_bond_count 4.0

error_branch 3.0

error_atom_count 1.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 65.00%, Predict selfies valid: 96.00%, error_type

NO 65.0

error_atom_seq 15.0

error_ring 9.0

error_bond_seq 7.0

error_branch 2.0

error_atom_count 1.0

error_bond_count 1.0

  

### epoch=4

  1. 评估长度4: 
     1. Correct: 89.33%, Predict selfies valid: 98.67%, error_type

NO 89.333333

error_bond_count 4.000000

error_atom_seq 4.000000

error_atom_count 1.333333

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 82.00%, Predict selfies valid: 95.00%, error_type

NO 82.0

error_ring 4.0

error_branch 3.0

error_bond_count 3.0

error_atom_seq 2.0

error_atom_count 2.0

error_bond_seq 2.0

error_other 1.0

1.0

  3. 评估长度6: 
     1. Correct: 58.00%, Predict selfies valid: 92.00%, error_type

NO 58.0

error_atom_seq 17.0

error_ring 9.0

error_bond_count 8.0

error_atom_count 5.0

error_branch 2.0

error_bond_seq 1.0

### epoch-4, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 62.67%, Predict selfies valid: 93.33%, error_type

NO 62.666667

error_bond_count 16.000000

error_atom_seq 8.000000

error_bond_seq 6.666667

error_atom_count 5.333333

1.333333

  2. 评估长度5: 
     1. Correct: 68.00%, Predict selfies valid: 94.00%, error_type

NO 68.0

error_bond_count 14.0

error_atom_seq 7.0

error_ring 4.0

error_bond_seq 2.0

error_branch 2.0

2.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 42.00%, Predict selfies valid: 86.00%, error_type

NO 42.0

error_atom_seq 19.0

error_bond_seq 14.0

error_atom_count 11.0

error_ring 9.0

error_bond_count 3.0

error_branch 2.0

  

### epoch=6

  1. 评估长度4: 
     1. Correct: 84.00%, Predict selfies valid: 96.00%, error_type

NO 84.0

error_bond_count 8.0

error_atom_seq 8.0

  2. 评估长度5: 
     1. Correct: 79.00%, Predict selfies valid: 95.00%, error_type

NO 79.0

error_bond_count 6.0

error_ring 5.0

error_bond_seq 3.0

error_branch 3.0

error_atom_count 2.0

error_atom_seq 2.0

  3. 评估长度6: 
     1. Correct: 66.00%, Predict selfies valid: 92.00%, error_type

NO 66.0

error_ring 9.0

error_atom_seq 7.0

error_bond_count 6.0

error_atom_count 6.0

error_bond_seq 4.0

error_branch 2.0

  

### epoch-6, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 81.33%, Predict selfies valid: 96.00%, error_type

NO 81.333333

error_bond_count 8.000000

error_atom_seq 8.000000

error_atom_count 1.333333

error_bond_seq 1.333333

  2. 评估长度5: 
     1. Correct: 77.00%, Predict selfies valid: 95.00%, error_type

NO 77.0

error_bond_count 8.0

error_ring 5.0

error_atom_seq 4.0

error_branch 3.0

error_bond_seq 2.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 59.00%, Predict selfies valid: 94.00%, error_type

NO 59.0

error_atom_seq 10.0

error_bond_count 9.0

error_ring 9.0

error_atom_count 7.0

error_bond_seq 4.0

error_branch 2.0

  

### epoch=8

  1. 评估长度4: 
     1. Correct: 82.67%, Predict selfies valid: 96.00%, error_type

NO 82.666667

error_bond_count 9.333333

error_atom_seq 4.000000

error_atom_count 2.666667

error_bond_seq 1.333333

  2. 评估长度5: 
     1. Correct: 79.00%, Predict selfies valid: 93.00%, error_type

NO 79.0

error_atom_seq 5.0

error_bond_count 5.0

3.0

error_ring 3.0

error_bond_seq 2.0

error_branch 2.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 64.00%, Predict selfies valid: 91.00%, error_type

NO 64.0

error_atom_seq 14.0

error_ring 9.0

error_atom_count 5.0

error_bond_count 4.0

error_bond_seq 2.0

1.0

error_branch 1.0

  

### epoch-8, 两阶段评估 (先写selfies, 再修正):

  1. 评估长度4: 
     1. Correct: 84.00%, Predict selfies valid: 97.33%, error_type

NO 84.000000

error_bond_count 10.666667

error_bond_seq 2.666667

error_atom_seq 1.333333

error_atom_count 1.333333

  2. 评估长度5: 
     1. Correct: 78.00%, Predict selfies valid: 93.00%, error_type

NO 78.0

error_bond_seq 4.0

error_ring 4.0

error_bond_count 4.0

error_atom_seq 3.0

error_branch 3.0

error_atom_count 2.0

1.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 55.00%, Predict selfies valid: 90.00%, error_type

NO 55.0

error_atom_seq 20.0

error_ring 9.0

error_bond_seq 6.0

error_atom_count 4.0

error_bond_count 4.0

error_branch 2.0

结论: 

  - 比起基线, 并没有明显提升, error_atom_seq也没有明显优势
  - 对长度6的数据的预测, 最好的情况是 (epoch=3) 71%
  - 两阶段评估, 比起一阶段评估, 在某些epoch训练中, 会让准确率下降. (也就是说: 修正任务 "可能" 会让结果变得糟糕)

## 尝试增强修正任务的训练数据数量

训练下来, 在epoch=2时, 对长度6的预测达到最好 (71%), 主要是因为增加的数据量多, 所以达到最好效果的epoch提前. 

增加修正任务的数据量, 并不能提升效果

# 尝试增加一个填空任务

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 增加了对原子位置的填空任务 (填一个空)

  1. 开始训练, 混合数据: epoch+1
     1. 填空任务, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 识别任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
  2. 继续训练, 混合数据: epoch+1
     1. 填空任务, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)

  

### epoch-1

  1. 评估长度4: 
     1. Correct: 80.00%, Predict selfies valid: 98.67%, error_type

NO 78.666667

error_bond_count 12.000000

error_atom_seq 8.000000

error_bond_seq 1.333333

  2. 评估长度5: 
     1. Correct: 69.00%, Predict selfies valid: 94.00%, error_type

NO 69.0

error_atom_seq 15.0

error_ring 5.0

error_branch 3.0

error_bond_count 3.0

error_bond_seq 3.0

error_atom_count 2.0

  3. 评估长度6: 
     1. Correct: 45.00%, Predict selfies valid: 88.00%, error_type

NO 44.0

error_atom_seq 25.0

error_atom_count 10.0

error_ring 9.0

error_bond_count 7.0

error_branch 2.0

error_bond_seq 2.0

1.0

### epoch-2

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 98.67%, error_type

NO 85.333333

error_bond_count 8.000000

error_atom_seq 5.333333

error_bond_seq 1.333333

  2. 评估长度5: 
     1. Correct: 77.00%, Predict selfies valid: 100.00%, error_type

NO 77.0

error_bond_count 9.0

error_ring 5.0

error_atom_seq 4.0

error_branch 3.0

error_bond_seq 1.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 63.00%, Predict selfies valid: 92.00%, error_type

NO 62.0

error_atom_seq 14.0

error_ring 9.0

error_bond_count 8.0

error_bond_seq 5.0

error_branch 1.0

1.0

### epoch-3

  1. 评估长度4: 
     1. Correct: 90.67%, Predict selfies valid: 98.67%, error_type

NO 90.666667

error_bond_count 8.000000

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 79.00%, Predict selfies valid: 97.00%, error_type

NO 78.0

error_atom_seq 7.0

error_ring 5.0

error_bond_count 5.0

error_branch 2.0

error_other 1.0

error_bond_seq 1.0

1.0

  3. 评估长度6: 
     1. Correct: 65.00%, Predict selfies valid: 92.00%, error_type

NO 62.0

error_bond_count 10.0

error_atom_seq 10.0

error_ring 9.0

error_bond_seq 4.0

error_atom_count 3.0

error_branch 2.0

### epoch-4

  1. 评估长度4: 
     1. Correct: 90.67%, Predict selfies valid: 97.33%, error_type

NO 89.333333

error_bond_count 6.666667

error_atom_seq 2.666667

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 86.00%, Predict selfies valid: 93.00%, error_type

NO 86.0

error_ring 5.0

error_branch 3.0

error_bond_count 2.0

error_bond_seq 2.0

error_atom_count 1.0

error_atom_seq 1.0

  3. 评估长度6: 
     1. Correct: 66.00%, Predict selfies valid: 90.00%, error_type

NO 65.0

error_atom_seq 12.0

error_ring 9.0

error_bond_count 6.0

error_bond_seq 4.0

error_branch 2.0

error_atom_count 2.0

  

### epoch-6

  1. 评估长度4: 
     1. Correct: 88.00%, Predict selfies valid: 98.67%, error_type

NO 88.000000

error_atom_seq 6.666667

error_bond_count 5.333333

Name: proportion, dtype: float64

Correct: 66 / 75 = 88.0 %

  2. 评估长度5: 
     1. Correct: 82.00%, Predict selfies valid: 94.00%, error_type

NO 82.0

error_bond_count 6.0

error_ring 4.0

error_atom_seq 3.0

error_branch 3.0

error_bond_seq 1.0

1.0

  3. 评估长度6: 
     1. Correct: 66.00%, Predict selfies valid: 90.00%, error_type

NO 66.0

error_atom_seq 13.0

error_ring 9.0

error_atom_count 5.0

error_bond_count 3.0

error_bond_seq 2.0

error_branch 2.0

  

### epoch-8

  1. 评估长度4: 
     1. Correct: 88.00%, Predict selfies valid: 97.33%, error_type

NO 88.0

error_bond_count 8.0

error_atom_seq 4.0

  2. 评估长度5: 
     1. Correct: 81.00%, Predict selfies valid: 93.00%, error_type

NO 81.0

error_bond_count 5.0

error_ring 4.0

error_branch 3.0

error_other 3.0

error_bond_seq 2.0

1.0

error_atom_seq 1.0

  3. 评估长度6: 
     1. Correct: 73.00%, Predict selfies valid: 95.00%, error_type

NO 73.0

error_atom_seq 10.0

error_ring 9.0

error_bond_count 4.0

error_bond_seq 2.0

error_branch 2.0

error_branch 2.0

  

对比基线, 并没有明显变化. 对长度6的预测, 最好的情况出现在epoch=8 (看起来像是个随机偏差)

  

  

# 使用填空任务+课程学习

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 增加了对原子位置的填空任务, 并使用课程学习

  1. 开始训练, 混合数据: 
     1. 填空任务, 长度2数据 (1个空), 长度3数据 (1个空), 长度4数据 (1个空), 长度5数据 (1个空)
  2. 继续训练, 混合数据: 
     1. 填空任务, 长度3数据 (2个空), 长度4数据 (2个空), 长度5数据 (2个空)
  3. 继续训练, 混合数据: 
     1. 填空任务, 长度4数据 (3个空), 长度5数据 (3个空)
  4. 继续训练, 混合数据: 
     1. 填空任务, 长度5数据 (4个空)
  5. 继续训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
  6. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)

### epoch=1

  1. 评估长度4: 
     1. Correct: 68.00%, Predict selfies valid: 93.33%, error_type

NO 66.666667

error_bond_count 16.000000

error_atom_seq 12.000000

error_bond_seq 2.666667

error_atom_count 2.666667

  2. 评估长度5: 
     1. Correct: 60.00%, Predict selfies valid: 92.00%, error_type

NO 60.0

error_bond_count 13.0

error_atom_seq 8.0

error_atom_count 6.0

error_ring 5.0

error_bond_seq 4.0

error_branch 3.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 37.00%, Predict selfies valid: 82.00%, error_type

NO 36.0

error_atom_seq 25.0

error_bond_seq 10.0

error_bond_count 10.0

error_ring 9.0

error_atom_count 8.0

error_branch 2.0

  

### epoch=2

  1. 评估长度4: 
     1. Correct: 89.33%, Predict selfies valid: 96.00%, error_type

NO 89.333333

error_atom_seq 5.333333

error_bond_count 5.333333

  2. 评估长度5: 
     1. Correct: 82.00%, Predict selfies valid: 93.00%, error_type

NO 81.0

error_atom_seq 7.0

error_ring 5.0

error_branch 3.0

error_bond_count 3.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 55.00%, Predict selfies valid: 93.00%, error_type

NO 55.0

error_atom_seq 17.0

error_ring 9.0

error_bond_count 8.0

error_atom_count 4.0

error_bond_seq 4.0

error_branch 2.0

error_other 1.0

### epoch=3

  1. 评估长度4: 
     1. Correct: 92.00%, Predict selfies valid: 98.67%, error_type

NO 92.000000

error_bond_count 4.000000

error_bond_seq 1.333333

error_other 1.333333

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 77.00%, Predict selfies valid: 93.00%, error_type

NO 77.0

error_atom_seq 12.0

error_ring 4.0

error_branch 3.0

error_bond_count 3.0

1.0

  3. 评估长度6: 
     1. Correct: 65.00%, Predict selfies valid: 94.00%, error_type

NO 64.0

error_atom_seq 16.0

error_ring 9.0

error_bond_seq 4.0

error_atom_count 3.0

error_branch 2.0

error_bond_count 2.0

  

对比基线, 没有明显提升

  

## 只使用填空任务+课程学习, 不使用最终任务训练, 评估最终任务试试:

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 只使用对原子位置的填空任务, 并使用课程学习

  1. 开始训练, 混合数据: 
     1. 填空任务, 长度2数据 (1个空), 长度3数据 (1个空), 长度4数据 (1个空), 长度5数据 (1个空)
  2. 继续训练, 混合数据: 
     1. 填空任务, 长度3数据 (2个空), 长度4数据 (2个空), 长度5数据 (2个空)
  3. 继续训练, 混合数据: 
     1. 填空任务, 长度4数据 (3个空), 长度5数据 (3个空)
  4. 继续训练, 混合数据: 
     1. 填空任务, 长度5数据 (4个空)

  

### epoch=1

  1. 评估长度4: 
     1. Correct: 52.00%, Predict selfies valid: 92.00%, error_type

NO 49.333333

error_bond_count 26.666667

error_atom_count 9.333333

error_atom_seq 8.000000

error_bond_seq 5.333333

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 45.00%, Predict selfies valid: 85.00%, error_type

NO 44.0

error_atom_seq 17.0

error_atom_count 14.0

error_bond_count 12.0

error_ring 5.0

error_bond_seq 4.0

error_branch 3.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 30.00%, Predict selfies valid: 80.00%, error_type

NO 30.0

error_atom_seq 27.0

error_bond_count 15.0

error_atom_count 14.0

error_ring 9.0

error_bond_seq 3.0

error_branch 2.0

  

### epoch=2

  1. 评估长度4: 
     1. Correct: 70.67%, Predict selfies valid: 94.67%, error_type

NO 70.666667

error_bond_count 14.666667

error_atom_seq 10.666667

error_bond_seq 2.666667

error_atom_count 1.333333

  2. 评估长度5: 
     1. Correct: 72.00%, Predict selfies valid: 95.00%, error_type

NO 72.0

error_atom_seq 11.0

error_ring 5.0

error_bond_count 3.0

error_atom_count 3.0

error_branch 3.0

error_bond_seq 2.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 48.00%, Predict selfies valid: 83.00%, error_type

NO 48.0

error_atom_seq 17.0

error_atom_count 12.0

error_ring 9.0

error_bond_seq 6.0

error_bond_count 4.0

error_branch 2.0

2.0

  

### epoch=3

  1. 评估长度4: 
     1. Correct: 45.33%, Predict selfies valid: 94.67%, error_type

NO 41.333333

error_atom_count 36.000000

error_bond_count 17.333333

2.666667

error_atom_seq 2.666667

  2. 评估长度5: 
     1. Correct: 75.00%, Predict selfies valid: 98.00%, error_type

NO 74.0

error_atom_seq 10.0

error_atom_count 5.0

error_ring 5.0

error_bond_count 3.0

error_branch 3.0

  3. 评估长度6: 
     1. Correct: 36.00%, Predict selfies valid: 91.00%, error_type

error_atom_count 33.0

NO 33.0

error_atom_seq 13.0

error_ring 9.0

error_bond_count 5.0

error_bond_seq 3.0

error_branch 2.0

2.0

比起epoch=2, 长度4/6的准确率下降, 因为最后一个环节是长度5的训练, epoch过高, 导致遗忘

  

  

## 调整一下训练计划:

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 只使用对原子位置的填空任务, 并使用课程学习

  1. 开始训练, 混合数据: 
     1. 填空任务, 长度2数据 (1个空), 长度3数据 (1个空), 长度4数据 (1个空), 长度5数据 (1个空), 没有增强数据
  2. 继续训练, 混合数据: 
     1. 填空任务, 长度3数据 (2个空), 长度4数据 (2个空), 长度5数据 (2个空), 长度5数据 (3个空), 没有增强数据
  3. 继续训练, 混合数据: 
     1. 填空任务, 长度2数据 (1个空), 长度3数据 (2个空), 长度4数据 (3个空), 长度5数据 (4个空), 没有增强数据

### epoch=1

  1. 评估长度4: 
     1. Correct: 58.67%, Predict selfies valid: 92.00%, error_type

NO 58.666667

error_bond_count 16.000000

error_atom_seq 14.666667

error_bond_seq 8.000000

error_atom_count 2.666667

  2. 评估长度5: 
     1. Correct: 48.00%, Predict selfies valid: 85.00%, error_type

NO 48.0

error_atom_seq 26.0

error_bond_count 12.0

error_ring 5.0

error_branch 3.0

error_bond_seq 3.0

error_atom_count 2.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 26.00%, Predict selfies valid: 91.00%, error_type

error_atom_seq 36.0

NO 26.0

error_bond_count 12.0

error_bond_seq 9.0

error_ring 9.0

error_atom_count 6.0

error_branch 2.0

### epoch=2

  1. 评估长度4: 
     1. Correct: 73.33%, Predict selfies valid: 90.67%, error_type

NO 72.000000

error_bond_count 9.333333

error_atom_seq 9.333333

error_bond_seq 5.333333

error_other 2.666667

error_atom_count 1.333333

  2. 评估长度5: 
     1. Correct: 69.00%, Predict selfies valid: 90.00%, error_type

NO 68.0

error_atom_seq 10.0

error_bond_count 8.0

error_ring 5.0

error_branch 3.0

error_atom_count 2.0

error_other 2.0

1.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 37.00%, Predict selfies valid: 82.00%, error_type

error_atom_count 33.0

NO 31.0

error_atom_seq 16.0

error_ring 9.0

error_bond_count 5.0

error_bond_seq 3.0

error_branch 2.0

1.0

### epoch=3

  1. 评估长度4: 
     1. Correct: 85.33%, Predict selfies valid: 97.33%, error_type

NO 85.333333

error_bond_count 10.666667

error_atom_seq 2.666667

error_bond_seq 1.333333

  2. 评估长度5: 
     1. Correct: 83.00%, Predict selfies valid: 96.00%, error_type

NO 82.0

error_ring 5.0

error_bond_count 4.0

error_branch 3.0

error_atom_seq 2.0

error_other 2.0

error_bond_seq 1.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 56.00%, Predict selfies valid: 90.00%, error_type

NO 56.0

error_atom_seq 12.0

error_bond_count 9.0

error_ring 7.0

error_bond_seq 6.0

error_atom_count 6.0

error_branch 2.0

2.0

  

对长度6的预测率下降.

  

  

## 再调整一下训练计划:

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 只使用对原子位置的填空任务, 并使用课程学习

  1. 开始训练, 混合数据: 
     1. 填空任务, 长度2数据 (1个空), 长度3数据 (1个空), 长度4数据 (1个空), 长度3数据 (2个空), 长度4数据 (2个空), 没有增强数据
  2. 继续训练, 混合数据: 
     1. 填空任务, 长度2数据 (1个空), 长度3数据 (1个空), 长度4数据 (1个空), 长度3数据 (2个空), 长度4数据 (2个空), 长度4数据 (3个空), 长度5数据 (1个空), 长度5数据 (2个空), 长度5数据 (3个空), 没有增强数据

  

### epoch=1

  1. 评估长度4: 
     1. Correct: 73.33%, Predict selfies valid: 96.00%, error_type

NO 72.0

error_atom_seq 16.0

error_bond_count 12.0

  2. 评估长度5: 
     1. Correct: 54.00%, Predict selfies valid: 89.00%, error_type

NO 54.0

error_atom_seq 17.0

error_bond_count 11.0

error_ring 5.0

4.0

error_bond_seq 3.0

error_atom_count 3.0

error_branch 2.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 27.00%, Predict selfies valid: 78.00%, error_type

error_atom_seq 29.0

NO 27.0

error_bond_count 14.0

10.0

error_ring 8.0

error_bond_seq 6.0

error_atom_count 4.0

error_branch 2.0

### epoch=2

  1. 评估长度4: 
     1. Correct: 85.33%, Predict selfies valid: 97.33%, error_type

NO 85.333333

error_bond_count 9.333333

error_atom_seq 5.333333

  2. 评估长度5: 
     1. Correct: 72.00%, Predict selfies valid: 91.00%, error_type

NO 71.0

error_bond_count 9.0

error_atom_seq 7.0

error_ring 4.0

error_branch 3.0

error_bond_seq 3.0

error_atom_count 2.0

1.0

  3. 评估长度6: 
     1. Correct: 57.00%, Predict selfies valid: 83.00%, error_type

NO 56.0

error_atom_seq 18.0

error_bond_count 9.0

error_ring 9.0

error_bond_seq 4.0

error_branch 2.0

error_atom_count 2.0

### epoch=3

  1. 评估长度4: 
     1. Correct: 89.33%, Predict selfies valid: 97.33%, error_type

NO 89.333333

error_bond_count 6.666667

error_atom_seq 2.666667

error_bond_seq 1.333333

  2. 评估长度5: 
     1. Correct: 71.00%, Predict selfies valid: 96.00%, error_type

NO 71.0

error_atom_seq 10.0

error_bond_count 6.0

error_ring 5.0

error_branch 3.0

error_bond_seq 2.0

error_atom_count 2.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 52.00%, Predict selfies valid: 84.00%, error_type

NO 48.0

error_atom_seq 25.0

error_ring 9.0

error_bond_count 9.0

error_atom_count 4.0

error_bond_seq 3.0

error_branch 2.0

  

结论: 

  - 只使用填空任务, 是可以训练最终任务的效果的, 但其上限略低于直接用最终任务训练  

  - 只使用填空任务, 并使用最终任务作为eval集, eval的结果与最终评估效果无正相关关系

  

### 额外: 在只训练 填空任务时, 用最终任务的验证集

以epoch=1为例: 

  - 第一轮: {'eval_loss': 0.35867372, 'eval_acc': 0.88223749, 'eval_runtime': 4.0369, 'eval_samples_per_second': 14.615, 'eval_steps_per_second': 7.432, 'epoch': 1.0, 'global_step/max_steps': '27/27', 'percentage': '100.00%', 'elapsed_time': '1m 28s', 'remaining_time': '0s'}
  - 第二轮: {'eval_loss': 0.56291121, 'eval_acc': 0.90578999, 'eval_runtime': 4.0042, 'eval_samples_per_second': 14.735, 'eval_steps_per_second': 7.492, 'epoch': 1.28, 'global_step/max_steps': '127/127', 'percentage': '100.00%', 'elapsed_time': '5m 29s', 'remaining_time': '0s'}

随着数据量增加, 最终任务的loss在增加, 并不收敛

  

以epoch=2为例: 

  - 第一轮: {'eval_loss': 0.18118805, 'eval_acc': 0.94013739, 'eval_runtime': 4.0072, 'eval_samples_per_second': 14.723, 'eval_steps_per_second': 7.486, 'epoch': 2.0, 'global_step/max_steps': '54/54', 'percentage': '100.00%', 'elapsed_time': '2m 52s', 'remaining_time': '0s'}
  - 第二轮: {'eval_loss': 0.53565764, 'eval_acc': 0.9185476, 'eval_runtime': 4.0818, 'eval_samples_per_second': 14.454, 'eval_steps_per_second': 7.35, 'epoch': 2.55, 'global_step/max_steps': '254/254', 'percentage': '100.00%', 'elapsed_time': '10m 54s', 'remaining_time': '0s'}

随着数据量增加, 最终任务的loss在增加, 并不收敛

  

以epoch=3为例: 

  - 第一轮: {'eval_loss': 0.77807945, 'eval_acc': 0.87634936, 'eval_runtime': 4.0158, 'eval_samples_per_second': 14.692, 'eval_steps_per_second': 7.47, 'epoch': 3.0, 'global_step/max_steps': '81/81', 'percentage': '100.00%', 'elapsed_time': '4m 16s', 'remaining_time': '0s'}
  - 第二轮: {'eval_loss': 0.53671777, 'eval_acc': 0.92541708, 'eval_runtime': 3.9615, 'eval_samples_per_second': 14.893, 'eval_steps_per_second': 7.573, 'epoch': 3.83, 'global_step/max_steps': '381/381', 'percentage': '100.00%', 'elapsed_time': '16m 19s', 'remaining_time': '0s'}

随着数据量增加, 最终任务的loss在增加, 并不收敛

验证集的loss, 与评估结果并没有正相关 关系

# 对只训练填空任务的讨论

发现如果只训练填空任务, 那么识别任务的准确率也会提升, epoch足够大的话, 准确率会接近对识别任务的训练. 

怀疑: 模型只提取了输入的图片和输出的selfies之间的关系, 而忽略的提示词中的指令

验证: 修改填空任务的输出, 让填空任务只输出BLANK应该填写的信息, 而不包括完整的selfies

# 使用填空任务+修正任务

训练的结果跟填空任务非常相似: 对长度6的准确率最好的情况是72%. 

看上去修正任务没有帮助, 或者命中了上一节中的怀疑
