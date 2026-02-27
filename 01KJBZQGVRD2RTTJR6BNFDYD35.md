---
title: 20250113 - 使用ms-swift对Qwen2-VL进行微调 [18] - 使用数据增强
confluence_page_id: 3343833
created_at: 2025-01-13T15:31:51+00:00
updated_at: 2025-01-16T11:09:01+00:00
---

# 

# 使用一倍增强数据 - 基线

见[20250111 - 使用ms-swift对Qwen2-VL进行微调 [18] - 使用相似COT流程进行两种互补任务]中的基线

#   

# 使用两倍增强数据 - 基线

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带两倍数据增强, 最多取100个), 长度4数据 (带两倍数据增强, 取全量)
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带两倍数据增强, 最多取100个), 长度5数据 (带两倍数据增强, 取全量)

  

### epoch=1

  1. 评估长度4: 
     1. Correct: 66.67%, Predict selfies valid: 86.67%, error_type

NO 66.666667

error_bond_count 12.000000

error_atom_seq 10.666667

8.000000

error_bond_seq 2.666667

  2. 评估长度5: 
     1. Correct: 61.00%, Predict selfies valid: 90.00%, error_type

NO 60.0

error_atom_count 14.0

error_bond_count 7.0

error_atom_seq 6.0

error_ring 5.0

3.0

error_branch 3.0

error_bond_seq 2.0

  3. 评估长度6: 
     1. Correct: 41.00%, Predict selfies valid: 83.00%, error_type

NO 40.0

error_atom_seq 14.0

error_bond_count 12.0

11.0

error_atom_count 8.0

error_ring 7.0

error_bond_seq 6.0

error_branch 2.0

  

### epoch=2

  1. 评估长度4: 
     1. Correct: 92.00%, Predict selfies valid: 97.33%, error_type

NO 92.000000

error_atom_seq 4.000000

error_bond_count 2.666667

1.333333

  2. 评估长度5: 
     1. Correct: 77.00%, Predict selfies valid: 93.00%, error_type

NO 77.0

error_atom_seq 6.0

error_ring 5.0

error_bond_count 4.0

error_atom_count 4.0

error_branch 3.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 67.00%, Predict selfies valid: 88.00%, error_type

NO 67.0

error_ring 9.0

error_atom_seq 8.0

error_atom_count 5.0

error_bond_seq 5.0

error_bond_count 4.0

error_branch 2.0

  

### epoch=3

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 100.00%, error_type

NO 86.666667

error_bond_count 6.666667

error_atom_seq 5.333333

error_atom_count 1.333333

  2. 评估长度5: 
     1. Correct: 77.00%, Predict selfies valid: 97.00%, error_type

NO 77.0

error_atom_seq 6.0

error_bond_count 6.0

error_ring 5.0

error_branch 3.0

error_atom_count 2.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 62.00%, Predict selfies valid: 92.00%, error_type

NO 62.0

error_atom_count 12.0

error_ring 9.0

error_atom_seq 8.0

error_bond_seq 4.0

error_bond_count 3.0

error_branch 2.0

  

### epoch=4

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 98.67%, error_type

NO 86.666667

error_bond_count 9.333333

error_other 1.333333

error_atom_seq 1.333333

1.333333

  2. 评估长度5: 
     1. Correct: 79.00%, Predict selfies valid: 92.00%, error_type

NO 79.0

error_bond_count 6.0

error_ring 5.0

error_bond_seq 3.0

2.0

error_branch 2.0

error_atom_seq 2.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 65.00%, Predict selfies valid: 93.00%, error_type

NO 65.0

error_atom_seq 12.0

error_ring 9.0

error_bond_count 6.0

error_bond_seq 4.0

error_atom_count 2.0

error_branch 2.0

### epoch=6

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 98.67%, error_type

NO 86.666667

error_bond_count 12.000000

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 76.00%, Predict selfies valid: 96.00%, error_type

NO 76.0

error_bond_count 8.0

error_atom_seq 4.0

error_ring 4.0

error_branch 3.0

error_atom_count 2.0

error_bond_seq 1.0

1.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 61.00%, Predict selfies valid: 91.00%, error_type

NO 60.0

error_bond_count 10.0

error_ring 9.0

error_atom_seq 9.0

error_atom_count 8.0

error_branch 2.0

error_bond_seq 1.0

1.0

  

结论: 比起使用一倍增强数据的基线, 准确率并没有太大变化

# 使用两倍增强数据 - 只使用填空任务

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 只使用对原子位置的填空任务, 并使用课程学习

  1. 开始训练, 混合数据: 
     1. 填空任务, 长度2数据 (1个空), 长度3数据 (1个空), 长度4数据 (1个空), 长度3数据 (2个空), 长度4数据 (2个空), 2倍增强数据
  2. 继续训练, 混合数据: 
     1. 填空任务, 长度2数据 (1个空), 长度3数据 (1个空), 长度4数据 (1个空), 长度3数据 (2个空), 长度4数据 (2个空), 长度4数据 (3个空), 长度5数据 (1个空), 长度5数据 (2个空), 长度5数据 (3个空), 2倍增强数据

### epoch=1

  1. 评估长度4: 
     1. Correct: 68.00%, Predict selfies valid: 94.67%, error_type

NO 68.000000

error_bond_count 16.000000

error_atom_seq 14.666667

error_bond_seq 1.333333

  2. 评估长度5: 
     1. Correct: 65.00%, Predict selfies valid: 96.00%, error_type

NO 64.0

error_atom_seq 13.0

error_bond_count 12.0

error_ring 5.0

error_branch 3.0

error_bond_seq 2.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 45.00%, Predict selfies valid: 84.00%, error_type

NO 43.0

error_atom_seq 20.0

error_bond_count 14.0

error_ring 9.0

error_bond_seq 6.0

error_atom_count 4.0

error_branch 2.0

2.0

  

### epoch=2

  1. 评估长度4: 
     1. Correct: 74.67%, Predict selfies valid: 92.00%, error_type

NO 74.666667

error_atom_seq 14.666667

error_bond_count 8.000000

error_atom_count 1.333333

1.333333

  2. 评估长度5: 
     1. Correct: 68.00%, Predict selfies valid: 90.00%, error_type

NO 68.0

error_atom_seq 19.0

error_ring 5.0

error_branch 3.0

error_atom_count 2.0

error_bond_seq 1.0

error_bond_count 1.0

1.0

  3. 评估长度6: 
     1. Correct: 51.00%, Predict selfies valid: 82.00%, error_type

NO 51.0

error_atom_seq 25.0

error_ring 9.0

error_bond_seq 8.0

error_atom_count 3.0

error_branch 2.0

error_bond_count 1.0

1.0

### epoch=3

  1. 评估长度4: 
     1. Correct: 81.33%, Predict selfies valid: 93.33%, error_type

NO 81.333333

error_bond_count 10.666667

error_atom_seq 5.333333

2.666667

  2. 评估长度5: 
     1. Correct: 84.00%, Predict selfies valid: 97.00%, error_type

NO 84.0

error_ring 5.0

error_atom_seq 4.0

error_branch 3.0

error_bond_count 3.0

1.0

  3. 评估长度6: 
     1. Correct: 61.00%, Predict selfies valid: 87.00%, error_type

NO 61.0

error_atom_seq 16.0

error_ring 7.0

error_bond_seq 5.0

4.0

error_atom_count 3.0

error_bond_count 2.0

error_branch 1.0

error_other 1.0

  

### epoch=4

  1. 评估长度4: 
     1. Correct: 80.00%, Predict selfies valid: 98.67%, error_type

NO 80.000000

error_bond_count 9.333333

error_atom_seq 8.000000

error_atom_count 2.666667

  2. 评估长度5: 
     1. Correct: 76.00%, Predict selfies valid: 93.00%, error_type

NO 76.0

error_atom_seq 9.0

error_bond_count 5.0

error_ring 5.0

error_branch 3.0

error_bond_seq 1.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 64.00%, Predict selfies valid: 93.00%, error_type

NO 63.0

error_atom_seq 16.0

error_ring 9.0

error_atom_count 5.0

error_bond_count 4.0

error_branch 2.0

error_bond_seq 1.0

  

结论: 对比极限, 长度4/5的准确率降低, 长度6的准确率差不多. 增强两倍数据 (对图形做简单变化) 并不能提升性能

# 使用一倍增强数据 - 基线

见[20250111 - 使用ms-swift对Qwen2-VL进行微调 [18] - 使用相似COT流程进行两种互补任务]中的基线

#   

# 使用两倍增强数据 - 基线

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带两倍数据增强, 最多取100个), 长度4数据 (带两倍数据增强, 取全量)
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带两倍数据增强, 最多取100个), 长度5数据 (带两倍数据增强, 取全量)

  

### epoch=1

  1. 评估长度4: 
     1. Correct: 66.67%, Predict selfies valid: 86.67%, error_type

NO 66.666667

error_bond_count 12.000000

error_atom_seq 10.666667

8.000000

error_bond_seq 2.666667

  2. 评估长度5: 
     1. Correct: 61.00%, Predict selfies valid: 90.00%, error_type

NO 60.0

error_atom_count 14.0

error_bond_count 7.0

error_atom_seq 6.0

error_ring 5.0

3.0

error_branch 3.0

error_bond_seq 2.0

  3. 评估长度6: 
     1. Correct: 41.00%, Predict selfies valid: 83.00%, error_type

NO 40.0

error_atom_seq 14.0

error_bond_count 12.0

11.0

error_atom_count 8.0

error_ring 7.0

error_bond_seq 6.0

error_branch 2.0

  

### epoch=2

  1. 评估长度4: 
     1. Correct: 92.00%, Predict selfies valid: 97.33%, error_type

NO 92.000000

error_atom_seq 4.000000

error_bond_count 2.666667

1.333333

  2. 评估长度5: 
     1. Correct: 77.00%, Predict selfies valid: 93.00%, error_type

NO 77.0

error_atom_seq 6.0

error_ring 5.0

error_bond_count 4.0

error_atom_count 4.0

error_branch 3.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 67.00%, Predict selfies valid: 88.00%, error_type

NO 67.0

error_ring 9.0

error_atom_seq 8.0

error_atom_count 5.0

error_bond_seq 5.0

error_bond_count 4.0

error_branch 2.0

  

### epoch=3

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 100.00%, error_type

NO 86.666667

error_bond_count 6.666667

error_atom_seq 5.333333

error_atom_count 1.333333

  2. 评估长度5: 
     1. Correct: 77.00%, Predict selfies valid: 97.00%, error_type

NO 77.0

error_atom_seq 6.0

error_bond_count 6.0

error_ring 5.0

error_branch 3.0

error_atom_count 2.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 62.00%, Predict selfies valid: 92.00%, error_type

NO 62.0

error_atom_count 12.0

error_ring 9.0

error_atom_seq 8.0

error_bond_seq 4.0

error_bond_count 3.0

error_branch 2.0

  

### epoch=4

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 98.67%, error_type

NO 86.666667

error_bond_count 9.333333

error_other 1.333333

error_atom_seq 1.333333

1.333333

  2. 评估长度5: 
     1. Correct: 79.00%, Predict selfies valid: 92.00%, error_type

NO 79.0

error_bond_count 6.0

error_ring 5.0

error_bond_seq 3.0

2.0

error_branch 2.0

error_atom_seq 2.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 65.00%, Predict selfies valid: 93.00%, error_type

NO 65.0

error_atom_seq 12.0

error_ring 9.0

error_bond_count 6.0

error_bond_seq 4.0

error_atom_count 2.0

error_branch 2.0

### epoch=6

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 98.67%, error_type

NO 86.666667

error_bond_count 12.000000

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 76.00%, Predict selfies valid: 96.00%, error_type

NO 76.0

error_bond_count 8.0

error_atom_seq 4.0

error_ring 4.0

error_branch 3.0

error_atom_count 2.0

error_bond_seq 1.0

1.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 61.00%, Predict selfies valid: 91.00%, error_type

NO 60.0

error_bond_count 10.0

error_ring 9.0

error_atom_seq 9.0

error_atom_count 8.0

error_branch 2.0

error_bond_seq 1.0

1.0

  

结论: 比起使用一倍增强数据的基线, 准确率并没有太大变化

# 使用两倍增强数据 - 只使用填空任务

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 只使用对原子位置的填空任务, 并使用课程学习

  1. 开始训练, 混合数据: 
     1. 填空任务, 长度2数据 (1个空), 长度3数据 (1个空), 长度4数据 (1个空), 长度3数据 (2个空), 长度4数据 (2个空), 2倍增强数据
  2. 继续训练, 混合数据: 
     1. 填空任务, 长度2数据 (1个空), 长度3数据 (1个空), 长度4数据 (1个空), 长度3数据 (2个空), 长度4数据 (2个空), 长度4数据 (3个空), 长度5数据 (1个空), 长度5数据 (2个空), 长度5数据 (3个空), 2倍增强数据

### epoch=1

  1. 评估长度4: 
     1. Correct: 68.00%, Predict selfies valid: 94.67%, error_type

NO 68.000000

error_bond_count 16.000000

error_atom_seq 14.666667

error_bond_seq 1.333333

  2. 评估长度5: 
     1. Correct: 65.00%, Predict selfies valid: 96.00%, error_type

NO 64.0

error_atom_seq 13.0

error_bond_count 12.0

error_ring 5.0

error_branch 3.0

error_bond_seq 2.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 45.00%, Predict selfies valid: 84.00%, error_type

NO 43.0

error_atom_seq 20.0

error_bond_count 14.0

error_ring 9.0

error_bond_seq 6.0

error_atom_count 4.0

error_branch 2.0

2.0

  

### epoch=2

  1. 评估长度4: 
     1. Correct: 74.67%, Predict selfies valid: 92.00%, error_type

NO 74.666667

error_atom_seq 14.666667

error_bond_count 8.000000

error_atom_count 1.333333

1.333333

  2. 评估长度5: 
     1. Correct: 68.00%, Predict selfies valid: 90.00%, error_type

NO 68.0

error_atom_seq 19.0

error_ring 5.0

error_branch 3.0

error_atom_count 2.0

error_bond_seq 1.0

error_bond_count 1.0

1.0

  3. 评估长度6: 
     1. Correct: 51.00%, Predict selfies valid: 82.00%, error_type

NO 51.0

error_atom_seq 25.0

error_ring 9.0

error_bond_seq 8.0

error_atom_count 3.0

error_branch 2.0

error_bond_count 1.0

1.0

### epoch=3

  1. 评估长度4: 
     1. Correct: 81.33%, Predict selfies valid: 93.33%, error_type

NO 81.333333

error_bond_count 10.666667

error_atom_seq 5.333333

2.666667

  2. 评估长度5: 
     1. Correct: 84.00%, Predict selfies valid: 97.00%, error_type

NO 84.0

error_ring 5.0

error_atom_seq 4.0

error_branch 3.0

error_bond_count 3.0

1.0

  3. 评估长度6: 
     1. Correct: 61.00%, Predict selfies valid: 87.00%, error_type

NO 61.0

error_atom_seq 16.0

error_ring 7.0

error_bond_seq 5.0

4.0

error_atom_count 3.0

error_bond_count 2.0

error_branch 1.0

error_other 1.0

  

### epoch=4

  1. 评估长度4: 
     1. Correct: 80.00%, Predict selfies valid: 98.67%, error_type

NO 80.000000

error_bond_count 9.333333

error_atom_seq 8.000000

error_atom_count 2.666667

  2. 评估长度5: 
     1. Correct: 76.00%, Predict selfies valid: 93.00%, error_type

NO 76.0

error_atom_seq 9.0

error_bond_count 5.0

error_ring 5.0

error_branch 3.0

error_bond_seq 1.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 64.00%, Predict selfies valid: 93.00%, error_type

NO 63.0

error_atom_seq 16.0

error_ring 9.0

error_atom_count 5.0

error_bond_count 4.0

error_branch 2.0

error_bond_seq 1.0

  

结论: 对比极限, 长度4/5的准确率降低, 长度6的准确率差不多. 增强两倍数据 (对图形做简单变化) 并不能提升性能

  

# 使用反转的selfies表达式进行数据增强

在之前的评估结果中, 发现很多预测的表达式和GT是相反顺序的, 比如 "[C][O][N]" 识别成了 "[N][O][C]". 

于是思考: 模型内在这些case上, 可能对于反转的selfies表达式更熟悉.

于是使用反转的selfies表达式, 生成了一套增强数据

数据生成脚本: [loop_selfies_and_image.generate_chain.CNO_count.reverse_selfies.cot.ipynb](/assets/01KJBZQGVRD2RTTJR6BNFDYD35/loop_selfies_and_image.generate_chain.CNO_count.reverse_selfies.cot.ipynb)

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量

  1. 开始训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
  2. 继续训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)

### epoch=1

  1. 评估长度4: 
     1. Correct: 86.67%, Predict selfies valid: 100.00%, error_type

NO 86.666667

error_bond_count 12.000000

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 72.00%, Predict selfies valid: 95.00%, error_type

NO 72.0

error_atom_seq 10.0

error_bond_count 7.0

error_ring 5.0

error_branch 3.0

error_atom_count 2.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 59.00%, Predict selfies valid: 92.00%, error_type

NO 58.0

error_atom_seq 12.0

error_bond_count 9.0

error_ring 9.0

error_bond_seq 6.0

2.0

error_atom_count 2.0

error_branch 1.0

error_other 1.0

  

### epoch=2

  1. 评估长度4: 
     1. Correct: 93.33%, Predict selfies valid: 98.67%, error_type

NO 93.333333

error_bond_count 2.666667

error_atom_seq 1.333333

error_bond_seq 1.333333

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 87.00%, Predict selfies valid: 98.00%, error_type

NO 87.0

error_ring 5.0

error_branch 3.0

error_bond_count 2.0

error_atom_count 2.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 74.00%, Predict selfies valid: 93.00%, error_type

NO 74.0

error_ring 9.0

error_atom_seq 6.0

error_bond_seq 4.0

error_bond_count 3.0

error_branch 2.0

error_atom_count 2.0

### epoch=3

  1. 评估长度4: 
     1. Correct: 97.33%, Predict selfies valid: 98.67%, error_type

NO 96.0

error_bond_count 4.0

  2. 评估长度5: 
     1. Correct: 88.00%, Predict selfies valid: 96.00%, error_type

NO 88.0

error_ring 5.0

error_bond_count 3.0

error_branch 3.0

error_atom_seq 1.0

  3. 评估长度6: 
     1. Correct: 79.00%, Predict selfies valid: 97.00%, error_type

NO 77.0

error_ring 9.0

error_bond_seq 5.0

error_atom_seq 4.0

error_bond_count 3.0

error_branch 2.0

  

### epoch=4

  1. 评估长度4: 
     1. Correct: 100.00%, Predict selfies valid: 100.00%, error_type

NO 100.0

  2. 评估长度5: 
     1. Correct: 91.00%, Predict selfies valid: 99.00%, error_type

NO 91.0

error_ring 4.0

error_branch 3.0

error_bond_count 1.0

1.0

  3. 评估长度6: 
     1. Correct: 78.00%, Predict selfies valid: 96.00%, error_type

NO 78.0

error_ring 9.0

error_atom_seq 5.0

error_bond_seq 4.0

error_branch 2.0

error_bond_count 1.0

error_other 1.0

  

  

最好的结果是79% (epoch=3), 长度4/5/6的准确率都提高

# 下一步: 解决ring的识别问题

发现用现有的随机算法, 生成Ring的时候, 形式单一

探索真实数据中的分布, 穷举selfies项目中的测试集, 使用如下代码: 

```
import pandas as pd
import selfies as sf

def load_csv(file_path, col):
    
    df = pd.read_csv(file_path)

    # 检查是否存在 "smiles" 列
    if "smiles" not in df.columns:
        print("Error: The column 'smiles' is not found in the CSV file.")
    else:
        # 遍历 "smiles" 列的内容
        for smiles in df[col]:
            try:
                selfies = sf.encoder(smiles)
                smiles_again = sf.decoder(selfies)
                if sf.len_selfies(selfies) < 7 and smiles == smiles_again and "[Ring1]" in selfies:
                    print(selfies)
            except Exception as e:
                pass

load_csv("/opt/huangyan/ms-swift/data_gen/selfies/tests/test_sets/qm9.csv", "smiles")
load_csv("/opt/huangyan/ms-swift/data_gen/selfies/tests/test_sets/molnet/freesolv.csv", "smiles")
load_csv("/opt/huangyan/ms-swift/data_gen/selfies/tests/test_sets/molnet/sider.csv", "smiles")
load_csv("/opt/huangyan/ms-swift/data_gen/selfies/tests/test_sets/molnet/tox21.csv", "smiles")
load_csv("/opt/huangyan/ms-swift/data_gen/selfies/tests/test_sets/molnet/toxcast.csv", "smiles")
``` 

输出: 

```
[C][C][C][Ring1][Ring1]
[C][C][O][Ring1][Ring1]
[C][C][C][C][Ring1][Ring1]
[C][C][C][O][Ring1][Ring1]
[C][N][C][C][Ring1][Ring1]
[O][C][C][C][Ring1][Ring1]
[C][C][C][C][Ring1][Ring2]
[C][C][O][C][Ring1][Ring2]
[C][C][N][C][Ring1][Ring2]
[C][=Branch1][Ring1][=C][/Cl][\Cl]
[C][C][C][Ring1][Ring1]
[C][Branch1][Ring1][C][O][O]
[C][Branch1][Ring1][C][Cl][Cl]
[C][Branch1][Ring1][C][Br][Cl]
[C][Branch1][Ring1][C][Br][Br]
[C][=Branch1][Ring1][=C][\Cl][\Cl]
[C][Branch1][Ring1][C][O][N]
[C][Branch1][Ring1][C][S][N]
[C][C][C][N][Ring1][Ring1]
[C][C][S][Ring1][Ring1]
[C][C][O][C][Ring1][Ring2]
[C][C][S][Ring1][Ring1]
[C][C][O][C][Ring1][Ring2]
[C][C][C][N][Ring1][Ring1]
``` 

从分布中看, 发现:

  - 对于小于6原子的分子, 都是以"[Ring1][Ring1]"结尾
  - 6原子的分子, 都是以 "[Ring1][Ring1]""[Ring1][Ring2]""[Branch1][Ring1]""[=Branch1][Ring1]"作为固定模式, 不会单独存在

使用固定模式来生成数据, 让每种模式的数量差不多. 

生成脚本: [loop_selfies_and_image.generate_chain.ring.generate_pic.ipynb](/assets/01KJBZQGVRD2RTTJR6BNFDYD35/loop_selfies_and_image.generate_chain.ring.generate_pic.ipynb)

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量

  1. 开始训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
     3. 识别任务, 包括ring的长度5的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度5的训练数据, 使用反转的selfies表达式进行的数据增强
  2. 继续训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     3. 识别任务, 包括ring的长度6的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度6的训练数据, 使用反转的selfies表达式进行的数据增强

注意: 长度4的原子是无法包含ring的, 所以ring的训练数据只能从长度5开始, 跟基础数据的长度不一样. 但为了验证其推理能力, 所以使用长度5+长度6的数据, 进行两步微调. 

所以, 只能等到测到长度7时, 才能准确的看到对于ring的数据的推理能力. 当前的实验, 只能粗浅地评估 ring的训练数据是否能影响训练效果 (但不包括推理能力)

### epoch=1

  1. 评估长度4: 
     1. Correct: 88.00%, Predict selfies valid: 96.00%, error_type

NO 88.000000

error_bond_count 8.000000

error_other 1.333333

error_ring 1.333333

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 79.00%, Predict selfies valid: 86.00%, error_type

NO 76.0

error_bond_count 9.0

error_atom_seq 8.0

error_atom_count 2.0

error_ring 2.0

error_bond_seq 1.0

error_branch 1.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 58.00%, Predict selfies valid: 84.00%, error_type

NO 56.0

error_atom_seq 24.0

error_bond_count 7.0

error_bond_seq 5.0

error_ring 3.0

error_branch 2.0

error_atom_count 2.0

error_other 1.0

  

### epoch=2

  1. 评估长度4: 
     1. Correct: 90.67%, Predict selfies valid: 98.67%, error_type

NO 90.666667

error_bond_count 6.666667

error_atom_seq 2.666667

  2. 评估长度5: 
     1. Correct: 88.00%, Predict selfies valid: 91.00%, error_type

NO 86.0

error_atom_seq 6.0

error_bond_count 3.0

error_ring 2.0

error_bond_seq 2.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 68.00%, Predict selfies valid: 88.00%, error_type

NO 67.0

error_atom_seq 20.0

error_bond_seq 4.0

error_bond_count 2.0

error_ring 2.0

error_atom_count 2.0

error_branch 2.0

error_other 1.0

### epoch=3

  1. 评估长度4: 
     1. Correct: 97.33%, Predict selfies valid: 98.67%, error_type

NO 97.333333

error_bond_count 1.333333

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 95.00%, Predict selfies valid: 97.00%, error_type

NO 94.0

error_branch 2.0

error_bond_count 1.0

error_atom_seq 1.0

error_bond_seq 1.0

error_ring 1.0

  3. 评估长度6: 
     1. Correct: 78.00%, Predict selfies valid: 89.00%, error_type

NO 75.0

error_atom_seq 16.0

error_ring 6.0

error_bond_seq 2.0

error_atom_count 1.0

  

长度6的评估文件: [evaluate_9.log](/assets/01KJBZQGVRD2RTTJR6BNFDYD35/evaluate_9.log)

### epoch=4

  1. 评估长度4: 
     1. Correct: 100.00%, Predict selfies valid: 100.00%, error_type

NO 100.0

  2. 评估长度5: 
     1. Correct: 89.00%, Predict selfies valid: 92.00%, error_type

NO 89.0

error_atom_seq 3.0

error_bond_count 3.0

error_ring 1.0

error_other 1.0

error_bond_seq 1.0

error_atom_count 1.0

error_branch 1.0

  3. 评估长度6: 
     1. Correct: 74.00%, Predict selfies valid: 86.00%, error_type

NO 72.0

error_atom_seq 19.0

error_ring 6.0

error_branch 3.0

### epoch=6

  1. 评估长度4: 
     1. Correct: 98.67%, Predict selfies valid: 100.00%, error_type

NO 98.666667

error_bond_count 1.333333

  2. 评估长度5: 
     1. Correct: 90.00%, Predict selfies valid: 97.00%, error_type

NO 90.0

error_bond_count 4.0

error_branch 3.0

error_atom_seq 2.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 20.00%, Predict selfies valid: 34.00%, error_type

error_ring 50.0

error_branch 19.0

NO 18.0

error_atom_seq 11.0

error_bond_seq 2.0

注意: 此处长度6的推理能力下降非常厉害, 模型大量将非环的分子 识别成了 带"[Ring1][Branch1]"的结构 (因为训练数据中没有长度6的普通分子, 而包含了长度6的带环分子, epoch调大以后, 模型将带环和长度6进行了绑定), 例如: 

```
                                                         Image Path                                        Predicted Predicted SMILES                                                                                                                                                                                                     GT   GT SMILES  Similarity  predict_selfies_valid?  error_ring?  error_branch?  error_atom_count?  error_atom_seq?  error_bond_count?  error_bond_seq?      error_type
0    /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_6/810.png      {"selfies": "[C][O][O][Ring1][Branch1][O]"}            C1OO1              {"single_bonds": 5, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 2, "O_count": 3, "N_count": 1, "branch_count": 0, "atom_count": 6, "selfies": "[C][O][O][C][O][N]"}      COOCON    0.000000                   False         True           True               True             True              False            False      error_ring
``` 

另外: "[Ring1][Branch1]"不应当有镜像结构, 其跟普通原子不同, 需要将镜像增强数据去掉

### 将ring相关的镜像增强数据去掉, 效果并没有提高

(结果略)

### 想法: 使用其他方式的表达式增强, 替代镜像增强

smiles表达式可以从mol随机某个起点开始生成, 转成selfies时也会不同, 比如: 

```
from rdkit import Chem
import selfies as sf

# 输入 SMILES 表达式
smiles_input = "C=C1NC1"

# 将 SMILES 转换为分子对象
mol = Chem.MolFromSmiles(smiles_input)

# 生成等价的随机 SMILES 表达式
for i in range(5):  # 生成 5 个随机等价的 SMILES
    random_smiles = Chem.MolToSmiles(mol, doRandom=True)
    selfies = sf.encoder(random_smiles)
    print(f"smiles={random_smiles}, selfies={selfies}")
``` 

输出: 

```
smiles=C1(=C)CN1, selfies=[C][=Branch1][C][=C][C][N][Ring1][Ring2]
smiles=C=C1CN1, selfies=[C][=C][C][N][Ring1][Ring1]
smiles=N1CC1=C, selfies=[N][C][C][Ring1][Ring1][=C]
smiles=C=C1CN1, selfies=[C][=C][C][N][Ring1][Ring1]
smiles=C1(NC1)=C, selfies=[C][Branch1][Branch1][N][C][Ring1][Ring1][=C]
``` 

在ring的数据上使用一下这种方案:

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 从长度5开始有ring的数据增强, 并为了长度6的ring数据增加单独的一轮混合数据

  1. 开始训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
  2. 继续训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     3. 识别任务, 包括ring的长度5的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度5的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)
  3. 继续训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     3. 识别任务, 包括ring的长度6的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度6的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)

### epoch=1

  1. 评估长度4: 
     1. Correct: 92.00%, Predict selfies valid: 100.00%, error_type

NO 92.000000

error_bond_count 6.666667

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 86.00%, Predict selfies valid: 89.00%, error_type

NO 83.0

error_atom_seq 7.0

error_bond_count 5.0

error_ring 3.0

error_atom_count 2.0

  3. 评估长度6: 
     1. Correct: 70.00%, Predict selfies valid: 89.00%, error_type

NO 70.0

error_atom_seq 13.0

error_bond_seq 6.0

error_bond_count 5.0

error_ring 4.0

error_branch 2.0

### epoch=2

  1. 评估长度4: 
     1. Correct: 98.67%, Predict selfies valid: 100.00%, error_type

NO 98.666667

error_bond_count 1.333333

  2. 评估长度5: 
     1. Correct: 93.00%, Predict selfies valid: 95.00%, error_type

NO 89.0

error_atom_seq 3.0

error_atom_count 2.0

error_branch 2.0

error_bond_count 2.0

error_bond_seq 1.0

error_ring 1.0

  3. 评估长度6: 
     1. Correct: 34.00%, Predict selfies valid: 68.00%, error_type

NO 31.0

error_atom_seq 20.0

error_ring 20.0

error_branch 12.0

error_atom_count 6.0

error_bond_count 5.0

4.0

error_bond_seq 2.0

长度6的准确率迅速下降, 错误地将很多普通分子识别了ring分子

### epoch=3

  1. 评估长度4: 
     1. Correct: 98.67%, Predict selfies valid: 100.00%, error_type

NO 98.666667

error_bond_count 1.333333

  2. 评估长度5: 
     1. Correct: 96.00%, Predict selfies valid: 93.00%, error_type

NO 93.0

error_bond_count 2.0

error_ring 2.0

error_atom_count 1.0

error_branch 1.0

error_atom_seq 1.0

  3. 评估长度6: 
     1. Correct: 80.00%, Predict selfies valid: 96.00%, error_type

NO 76.0

error_atom_seq 12.0

error_atom_count 3.0

error_branch 3.0

error_ring 3.0

error_bond_seq 2.0

error_other 1.0

### epoch=4

  1. 评估长度4: 
     1. Correct: 97.33%, Predict selfies valid: 97.33%, error_type

NO 94.666667

error_bond_count 5.333333

  2. 评估长度5: 
     1. Correct: 90.00%, Predict selfies valid: 91.00%, error_type

NO 88.0

error_bond_count 5.0

error_atom_seq 3.0

error_ring 2.0

error_atom_count 1.0

error_branch 1.0

  3. 评估长度6: 
     1. Correct: 74.00%, Predict selfies valid: 93.00%, error_type

NO 73.0

error_atom_seq 12.0

error_bond_seq 4.0

error_branch 3.0

error_ring 3.0

error_bond_count 3.0

error_atom_count 2.0

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

### epoch=1

  1. 评估长度4: 
     1. Correct: 85.33%, Predict selfies valid: 96.00%, error_type

NO 85.333333

error_bond_count 8.000000

error_atom_seq 2.666667

error_bond_seq 1.333333

error_atom_count 1.333333

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 71.00%, Predict selfies valid: 90.00%, error_type

NO 69.0

error_atom_seq 8.0

error_bond_seq 7.0

error_bond_count 6.0

error_atom_count 4.0

error_branch 3.0

error_other 2.0

1.0

  3. 评估长度6: 
     1. Correct: 66.00%, Predict selfies valid: 87.00%, error_type

NO 66.0

error_atom_seq 19.0

error_bond_seq 4.0

error_bond_count 3.0

error_ring 3.0

error_atom_count 3.0

error_branch 2.0

### epoch=2

  1. 评估长度4: 
     1. Correct: 90.67%, Predict selfies valid: 100.00%, error_type

NO 90.666667

error_bond_count 6.666667

error_atom_seq 2.666667

  2. 评估长度5: 
     1. Correct: 88.00%, Predict selfies valid: 97.00%, error_type

NO 84.0

error_bond_count 5.0

error_atom_seq 5.0

error_branch 3.0

error_bond_seq 2.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 71.00%, Predict selfies valid: 89.00%, error_type

NO 71.0

error_atom_seq 11.0

error_bond_count 6.0

error_atom_count 4.0

error_ring 4.0

error_branch 2.0

error_bond_seq 1.0

error_other 1.0

  

### epoch=3

  1. 评估长度4: 
     1. Correct: 98.67%, Predict selfies valid: 100.00%, error_type

NO 98.666667

error_bond_count 1.333333

  2. 评估长度5: 
     1. Correct: 94.00%, Predict selfies valid: 100.00%, error_type

NO 91.0

error_ring 3.0

error_atom_seq 3.0

error_bond_count 2.0

error_atom_count 1.0

  3. 评估长度6: 
     1. Correct: 82.00%, Predict selfies valid: 91.00%, error_type

NO 81.0

error_atom_seq 8.0

error_bond_count 3.0

error_ring 3.0

error_atom_count 2.0

error_branch 2.0

error_bond_seq 1.0

### epoch=4

  1. 评估长度4: 
     1. Correct: 100.00%, Predict selfies valid: 100.00%, error_type

NO 100.0

  2. 评估长度5: 
     1. Correct: 93.00%, Predict selfies valid: 100.00%, error_type

NO 92.0

error_ring 3.0

error_atom_seq 3.0

error_bond_count 2.0

  3. 评估长度6: 
     1. Correct: 84.00%, Predict selfies valid: 95.00%, error_type

NO 83.0

error_atom_seq 6.0

error_bond_seq 3.0

error_ring 3.0

error_branch 2.0

error_other 1.0

error_bond_count 1.0

error_atom_count 1.0

评估日志: [evaluate_24.log](/assets/01KJBZQGVRD2RTTJR6BNFDYD35/evaluate_24.log) 

对错误类型进行分析: 

```
[N][O][O][N][Ring1][Ring1], [N][O][N][O][Ring1][Ring2], 相邻原子交换 + Ring识别错误
[C][O][C][=C][Ring1][Ring1], [C][O][C][=C][Ring1][Ring2], Ring识别错误
[N][C][C][=N][=Ring1][Ring1], [O][=C][C][N][Ring1][Ring1], 原子类型错误(位置没错) + 键类型错误(位置没错)
[N][=C][O][C][C][=N], [C][=N][O][C][N][=C], 相邻原子交换*2
[O][C][C][C][Ring1][Ring1], [C][C][C][O][Ring1][Ring1], 局部反转
[C][#C][C][=C][C][N], [C][C][#C][C][=C][N], 键类型错误(位置没错)
[C][=C][C][=N][Ring1][Ring1], [C][=C][N][C][Ring1][Ring1], 相邻原子交换+键类型错误(位置没错)
[C][C][#C][=N][C][N], [C][C][#C][N][=C][N], 键类型错误(位置没错)
[O][=C][C][C][#C][C], [C][Branch1][Ring1][C][=O][#C], 没有识别出ring/branch
[O][C][C][C][#C][C], [C][Branch1][Ring1][C][#C][O], 没有识别出ring/branch
[O][=N][C][C][O][N], [O][N][=C][C][O][N], 键类型错误(位置没错)
[O][C][O][C][N][N], [O][C][O][N][C][N], 相邻原子交换
[C][O][C][C][N][N], [O][Branch1][C][C][C][N], 没有识别出ring/branch
[C][C][O][C][C][N], [N][C][O][C][C][C], 相邻原子交换 + 整体反转
[O][=C][C][C][C][C], [C][Branch1][C][C][C][=O], 没有识别出ring/branch
[N][=C][C][C][C][N], [C][Branch1][Ring1][C][N][=N], 没有识别出ring/branch
``` 

  

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 从长度5开始有ring的数据增强, 并为了长度6的ring数据增加单独的一轮轻度混合数据

  1. 开始训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-3数据 (最多取100个), 长度4数据 ( 取全量)
  2. 继续训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-4数据 (最多取100个), 长度5数据 ( 取全量)
     3. 识别任务, 包括ring的长度5的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度5的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)
  3. 继续训练, 混合数据: epoch+1
     1. 识别任务, 带图形变换的数据增强, 长度1-5数据, 最多取50个
     2. 识别任务, 使用反转的selfies表达式进行的数据增强, 长度1-5数据, 最多取50个
     3. 识别任务, 包括ring的长度6的训练数据, 带图形变换的数据增强
     4. 识别任务, 包括ring的长度6的训练数据, 使用smiles随机化进行的数据增强, 增强比例为2倍 (去重后为1.5倍左右)

  

### epoch=1

  1. 评估长度4: 
     1. Correct: 85.33%, Predict selfies valid: 93.33%, error_type

NO 85.333333

error_bond_count 6.666667

error_atom_seq 4.000000

error_atom_count 1.333333

error_ring 1.333333

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 74.00%, Predict selfies valid: 89.00%, error_type

NO 74.0

error_atom_seq 12.0

error_bond_count 6.0

error_ring 3.0

error_atom_count 2.0

error_bond_seq 1.0

error_other 1.0

1.0

  3. 评估长度6: 
     1. Correct: 49.00%, Predict selfies valid: 87.00%, error_type

NO 48.0

error_atom_seq 22.0

error_bond_count 13.0

error_atom_count 6.0

error_bond_seq 4.0

error_ring 3.0

error_branch 2.0

error_other 2.0

### epoch=2

  1. 评估长度4: 
     1. Correct: 90.67%, Predict selfies valid: 100.00%, error_type

NO 90.666667

error_bond_count 6.666667

error_atom_count 1.333333

error_atom_seq 1.333333

  2. 评估长度5: 
     1. Correct: 85.00%, Predict selfies valid: 94.00%, error_type

NO 83.0

error_atom_seq 7.0

error_bond_count 5.0

error_ring 3.0

error_atom_count 1.0

error_other 1.0

  3. 评估长度6: 
     1. Correct: 72.00%, Predict selfies valid: 87.00%, error_type

NO 69.0

error_atom_seq 12.0

error_atom_count 7.0

error_ring 5.0

error_bond_seq 3.0

error_other 2.0

error_bond_count 1.0

1.0

### epoch=3

  1. 评估长度4: 
     1. Correct: 96.00%, Predict selfies valid: 100.00%, error_type

NO 96.000000

error_atom_seq 1.333333

error_bond_count 1.333333

error_other 1.333333

  2. 评估长度5: 
     1. Correct: 93.00%, Predict selfies valid: 93.00%, error_type

NO 91.0

error_atom_seq 4.0

error_ring 3.0

error_atom_count 1.0

error_bond_seq 1.0

  3. 评估长度6: 
     1. Correct: 73.00%, Predict selfies valid: 95.00%, error_type

NO 71.0

error_atom_seq 10.0

error_atom_count 5.0

error_bond_count 5.0

error_branch 5.0

error_bond_seq 2.0

error_ring 1.0

error_other 1.0

### epoch=4

  1. 评估长度4: 
     1. Correct: 96.00%, Predict selfies valid: 97.33%, error_type

NO 96.000000

error_atom_seq 1.333333

error_bond_count 1.333333

1.333333

  2. 评估长度5: 
     1. Correct: 92.00%, Predict selfies valid: 88.00%, error_type

NO 91.0

error_atom_count 3.0

error_ring 3.0

error_atom_seq 2.0

error_bond_count 1.0

  3. 评估长度6: 
     1. Correct: 68.00%, Predict selfies valid: 91.00%, error_type

NO 65.0

error_atom_seq 14.0

error_bond_seq 7.0

error_ring 5.0

error_atom_count 4.0

error_branch 2.0

error_bond_count 2.0

error_other 1.0

  

目前最好的效果是对长度6的预测84%, 训练的方式是:

  1. 长度4/长度5 使用 标准数据+selfies反转的增强数据+ring相关的数据+ring相关的随机增强数据
  2. 长度6 使用 ring相关的数据+ring相关的随机增强数据

经验: 

  - ring相关的训练数据+随机增强数据, 可以增强对长度6中ring的识别 (ring从长度5开始才有)
  - 在最后阶段, 长度6只使用ring相关的数据训练, 并没有影响普通数据的推理能力, 但能提高长度6的评估结果

  

对错误的分析见SOTA的描述, 其指出了数据增强方向: 

  1. 相邻原子交换
  2. 键类型替换
  3. 增强ring/branch的例子  

  

  

# 想法

还有一种COT方式没有测试: 

  1. 对COT使用分步训练:
     1. 第一步：原子序列为 [C, C, O]
     2. 第二步：键类型为 [单键, 双键]
     3. 第三步：SELFIES 表达式为 [C][C][=O]
     4. 逐步验证结果COT:
        1. SELFIES: [C][=C][O]
        2. 问题：键类型是否正确？
        3. 检查：第2步，键类型为双键，检查是否符合化学规则。
        4. 修订：将 [C][=C][O] 修正为 [C][C][=O]。

# 下一步

目前认为: 只训练长度4/5的情况下, 对长度6的推理能力已经足够好了 (80%+), 现在向7原子进行推进

用当前最好的模型评估长度7的分子: 

Correct: 43.00%, Predict selfies valid: 85.00%, error_type  
NO 40.0  
error_atom_seq 28.0  
error_ring 9.0  
error_atom_count 7.0  
error_branch 6.0  
error_bond_seq 5.0  
error_bond_count 4.0  
error_other 1.0
