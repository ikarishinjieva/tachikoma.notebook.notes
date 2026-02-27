---
title: 20250108 - 使用ms-swift对Qwen2-VL进行微调 [17] - 增强COT流程
confluence_page_id: 3343749
created_at: 2025-01-08T05:11:28+00:00
updated_at: 2025-01-11T08:30:08+00:00
---

# 对前继情况进行整理

### 当前最优解1: 

增加键合检验COT任务, 进行课程训练

  1. 开始训练, 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量): epoch+4
  2. 继续训练, 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量): epoch+4
  3. 继续训练, 最终任务, 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量): epoch+4
  4. 继续训练, 最终任务, 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量): epoch+4
  5. 评估长度4: 
     1. Correct: 93.33%, Predict selfies valid Accuracy: 100.00%, Double Bonds Match Accuracy: 93.33%, Triple Bonds Match Accuracy: 98.67%

  6. 评估长度5: 
     1. Correct: 88.00%, Predict selfies valid Accuracy: 99.00%, Double Bonds Match Accuracy: 93.00%, Triple Bonds Match Accuracy: 95.00%

  7. 评估长度6: 
     1. Correct: 71.00%, Predict selfies valid Accuracy: 95.00%, Double Bonds Match Accuracy: 87.00%, Triple Bonds Match Accuracy: 93.00%

### 当前最优解2: 

增加键合检验COT任务, 使用混合数据增强理解

  1. 开始训练, 使用混合数据: epoch+4
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-3数据 (不带数据增强, 最多取100个), 长度4数据 (不带数据增强, 取全量)
  2. 继续训练, 使用混合数据: epoch+4
     1. 提示词中明确指定原子数, 使用COT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量)
     2. 键合检验任务, 使用COT流程, 长度1-4数据 (不带数据增强, 最多取100个), 长度5数据 (不带数据增强, 取全量)
  3. 评估长度4: 
     1. Correct: 92.00%, Predict selfies valid Accuracy: 100.00%, Double Bonds Match Accuracy: 93.33%, Triple Bonds Match Accuracy: 97.33%

  4. 评估长度5: 
     1. Correct: 87.00%, Predict selfies valid Accuracy: 95.00%, Double Bonds Match Accuracy: 94.00%, Triple Bonds Match Accuracy: 95.00%

  5. 评估长度6: 
     1. Correct: 71.00%, Predict selfies valid Accuracy: 94.00%, Double Bonds Match Accuracy: 90.00%, Triple Bonds Match Accuracy: 95.00%

### 当前基线:

只使用最终任务(CoT流程), 建立CoT的基线

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: Correct: 74.67%, Predict selfies valid Accuracy: 92.00%, Double Bonds Match Accuracy: 77.33%, Triple Bonds Match Accuracy: 84.00%
  4. 评估长度5: Correct: 64.00%, Predict selfies valid Accuracy: 90.00%, Double Bonds Match Accuracy: 78.00%, Triple Bonds Match Accuracy: 83.00%
  5. 评估长度6: Correct: 49.00%, Predict selfies valid Accuracy: 86.00%, Double Bonds Match Accuracy: 78.00%, Triple Bonds Match Accuracy: 91.00%

比起之前 非CoT流程(仅要求做键合检测)的基线, 准确率都提高10%左右, Predict selfies valid Accuracy等没有提高

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  3. 评估长度4: Correct: 82.67%, Predict selfies valid Accuracy: 97.33%, Double Bonds Match Accuracy: 85.33%, Triple Bonds Match Accuracy: 92.00%
  4. 评估长度5: Correct: 73.00%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 82.00%, Triple Bonds Match Accuracy: 89.00%
  5. 评估长度6: Correct: 61.00%, Predict selfies valid Accuracy: 92.00%, Double Bonds Match Accuracy: 85.00%, Triple Bonds Match Accuracy: 95.00%

比起之前 非CoT流程(仅要求做键合检测)的基线, 长度4/5的准确率提高不到5%, 长度6的准确率提高18% (Double Bonds Match Accuracy提高8%, 其他准确率提升不明显)

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  3. 评估长度4: Correct: 85.33%, Predict selfies valid Accuracy: 97.33%, Double Bonds Match Accuracy: 89.33%, Triple Bonds Match Accuracy: 94.67%
  4. 评估长度5: Correct: 84.00%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 95.00%, Triple Bonds Match Accuracy: 91.00%
  5. 评估长度6: Correct: 67.00%, Predict selfies valid Accuracy: 94.00%, Double Bonds Match Accuracy: 90.00%, Triple Bonds Match Accuracy: 95.00%

比起之前 非CoT流程(仅要求做键合检测)的基线, 长度4/5/5的准确率分别提高5%/10%/20%, Double Bonds Match Accuracy分别提高 3%/9%/13%, 其他准确率提升不明显

对比现在的最优解 (长度4/5/6准确率92%/87%/71%), 准确率下降3%-6%

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+8
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+8
  3. 评估长度4: Correct: 81.33%, Predict selfies valid Accuracy: 97.33%, Double Bonds Match Accuracy: 89.33%, Triple Bonds Match Accuracy: 96.00%
  4. 评估长度5: Correct: 80.00%, Predict selfies valid Accuracy: 96.00%, Double Bonds Match Accuracy: 90.00%, Triple Bonds Match Accuracy: 94.00%
  5. 评估长度6: Correct: 69.00%, Predict selfies valid Accuracy: 94.00%, Double Bonds Match Accuracy: 89.00%, Triple Bonds Match Accuracy: 97.00%

比起epoch=4, 没有提高

### 目前的结论

  - 目前效果最好的, 是使用 键合检验任务(COT流程) + 最终任务(COT流程), 无论是混合式训练或者课程训练, (epoch=4, 数据量相当于基线中的epoch=8) 对长度6的推理正确率提升到了71%
  - 单纯使用最终任务(COT流程), 基线比最优解稍低. 看上去: 键合检验任务(COT流程)并没有发挥太大作用 (Predict selfies valid Accuracy提升不多)
  - 下一步需要研究最优解后, 错误case出现在哪里

# 研究当前最优解, 错误case的分布

最优解的日志: 

  - 日志: *[evaluate_9.log](<http://8.134.54.170:8330/download/attachments/3343656/evaluate_9.log?version=1&modificationDate=1736264935000&api=v2>)*
  - 日志: *[evaluate_18.log](<http://8.134.54.170:8330/download/attachments/3343656/evaluate_18.log?version=1&modificationDate=1736264917000&api=v2>)*

将两份日志中的差异合并在一起进行统计: [chem.2.diff](/assets/01KJBZQ6DWHS6EB67JVEPB2H98/chem.2.diff)

统计错误类型:

```
tachikoma@TachikomadeMacBook-Pro ~ % cat /Users/tachikoma/Downloads/chem.2.diff | grep ANALYZE | sort | uniq -c
  17 ANALYZE: 双键/三键个数错误
   3 ANALYZE: 双键/三键位置错误
   4 ANALYZE: 识别错了Branch
  18 ANALYZE: 识别错了Ring
   7 ANALYZE: 元素个数错误
   9 ANALYZE: 元素排列错误
``` 

# 

# 修改evaluate脚本, 对各种错误进行评估

# 

错误分类: 

# 

ANALYZE: 识别错了Ring  
ANALYZE: 识别错了Branch  
ANALYZE: 元素个数错误  
ANALYZE: 元素排列错误  
ANALYZE: 双键/三键个数错误  
ANALYZE: 双键/三键位置错误

# 

脚本: [evaluate_loop_selfies.specific_atom_count.with_bond_count.cot.py](/assets/01KJBZQ6DWHS6EB67JVEPB2H98/evaluate_loop_selfies.specific_atom_count.with_bond_count.cot.py)

增加了对于错误率的判断, 需要重新生成基线

# 在新的评估指标上, 建立新的基线

只使用最终任务(CoT流程), 建立CoT的基线

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: 
     1. Correct: 74.67%, Predict selfies valid: 92.00%, Error ring: 0.00%, Error branch: 0.00%, Error atom_count: 4.00%, Error atom_seq: 10.67%, Error bond_count: 14.67%, Error bond_seq: 44.00%
     2. Correct: 74.67%, Predict selfies valid: 93.33%, error_type

NO 74.666667

error_bond_count 10.666667

error_atom_count 6.666667

error_atom_seq 5.333333

error_bond_seq 2.666667

  4. 评估长度5: 
     1. Correct: 60.00%, Predict selfies valid: 94.00%, Error ring: 5.00%, Error branch: 3.00%, Error atom_count: 17.00%, Error atom_seq: 25.00%, Error bond_count: 23.00%, Error bond_seq: 55.00%
     2. Correct: 63.00%, Predict selfies valid: 94.00%, error_type

NO 63.0

error_bond_count 10.0

error_atom_seq 10.0

error_bond_seq 5.0

error_ring 5.0

error_atom_count 4.0

error_branch 3.0

  5. 评估长度6: 
     1. Correct: 57.00%, Predict selfies valid: 91.00%, Error ring: 9.00%, Error branch: 5.00%, Error atom_count: 26.00%, Error atom_seq: 30.00%, Error bond_count: 19.00%, Error bond_seq: 61.00%
     2. Correct: 44.00%, Predict selfies valid: 85.00%, error_type

NO 44.0

error_atom_count 19.0

error_atom_seq 13.0

error_bond_seq 9.0

error_ring 9.0

error_bond_count 4.0

error_branch 2.0

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  3. 评估长度4: 
     1. Correct: 81.33%, Predict selfies valid: 96.00%, Error ring: 0.00%, Error branch: 0.00%, Error atom_count: 4.00%, Error atom_seq: 4.00%, Error bond_count: 10.67%, Error bond_seq: 48.00%

     2. Correct: 84.00%, Predict selfies valid: 96.00%, error_type

NO 84.000000

error_bond_count 10.666667

error_atom_count 4.000000

1.333333

  4. 评估长度5: 
     1. Correct: 75.00%, Predict selfies valid: 97.00%, Error ring: 5.00%, Error branch: 3.00%, Error atom_count: 12.00%, Error atom_seq: 16.00%, Error bond_count: 12.00%, Error bond_seq: 46.00%
     2. Correct: 78.00%, Predict selfies valid: 99.00%, error_type

NO 78.0

error_bond_count 8.0

error_ring 5.0

error_atom_count 3.0

error_branch 3.0

error_atom_seq 2.0

error_bond_seq 1.0

  5. 评估长度6: 
     1. Correct: 57.00%, Predict selfies valid: 90.00%, Error ring: 9.00%, Error branch: 5.00%, Error atom_count: 23.00%, Error atom_seq: 29.00%, Error bond_count: 18.00%, Error bond_seq: 59.00%
     2. Correct: 65.00%, Predict selfies valid: 92.00%, error_type

NO 64.0

error_atom_count 10.0

error_ring 9.0

error_bond_count 9.0

error_atom_seq 3.0

error_bond_seq 3.0

error_branch 2.0

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  3. 评估长度4: 
     1. Correct: 82.67%, Predict selfies valid: 96.00%, Error ring: 0.00%, Error branch: 0.00%, Error atom_count: 4.00%, Error atom_seq: 6.67%, Error bond_count: 9.33%, Error bond_seq: 46.67%
     2. Correct: 82.67%, Predict selfies valid: 98.67%, error_type

NO 82.666667

error_atom_count 9.333333

error_bond_count 5.333333

error_atom_seq 1.333333

error_bond_seq 1.333333

  4. 评估长度5: 
     1. Correct: 78.00%, Predict selfies valid: 98.00%, Error ring: 5.00%, Error branch: 3.00%, Error atom_count: 15.00%, Error atom_seq: 18.00%, Error bond_count: 9.00%, Error bond_seq: 41.00%
     2. Correct: 79.00%, Predict selfies valid: 95.00%, error_type

NO 79.0

error_atom_seq 6.0

error_bond_count 5.0

error_ring 4.0

error_branch 3.0

error_atom_count 1.0

1.0

error_bond_seq 1.0

  5. 评估长度6: 
     1. Correct: 66.00%, Predict selfies valid: 96.00%, Error ring: 9.00%, Error branch: 5.00%, Error atom_count: 23.00%, Error atom_seq: 27.00%, Error bond_count: 15.00%, Error bond_seq: 55.00%
     2. Correct: 63.00%, Predict selfies valid: 94.00%, error_type

NO 63.0

error_atom_count 14.0

error_ring 9.0

error_atom_seq 8.0

error_branch 2.0

error_bond_count 2.0

error_bond_seq 1.0

error_other 1.0

提示词中明确指定原子数, 使用CoT流程

  1. 开始训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+6
  2. 继续训练, 提示词中明确指定原子数, 使用CoT流程, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+6
  3. 评估长度4: 
     1. Correct: 96.00%, Predict selfies valid: 100.00%, Error ring: 0.00%, Error branch: 0.00%, Error atom_count: 1.33%, Error atom_seq: 1.33%, Error bond_count: 2.67%, Error bond_seq: 33.33%
     2. Correct: 80.00%, Predict selfies valid: 100.00%, error_type

NO 80.000000

error_bond_count 10.666667

error_atom_count 8.000000

error_atom_seq 1.333333

  4. 评估长度5: 
     1. Correct: 82.00%, Predict selfies valid: 96.00%, Error ring: 5.00%, Error branch: 4.00%, Error atom_count: 12.00%, Error atom_seq: 13.00%, Error bond_count: 5.00%, Error bond_seq: 44.00%
     2. Correct: 81.00%, Predict selfies valid: 98.00%, error_type

NO 81.0

error_ring 5.0

error_bond_count 4.0

error_branch 3.0

error_atom_count 3.0

error_atom_seq 2.0

error_bond_seq 1.0

error_other 1.0

  5. 评估长度6: 
     1. Correct: 68.00%, Predict selfies valid: 94.00%, Error ring: 9.00%, Error branch: 5.00%, Error atom_count: 18.00%, Error atom_seq: 23.00%, Error bond_count: 8.00%, Error bond_seq: 52.00%
     2. Correct: 59.00%, Predict selfies valid: 94.00%, error_type

NO 58.0

error_atom_count 21.0

error_ring 9.0

error_atom_seq 4.0

error_bond_seq 3.0

error_bond_count 3.0

error_branch 2.0

### 研究评估结果

所有评估结果中, 类似于: 

```
{"initial_selfies": "[O][O][=C][=C][C][=C]", "validation": "The bonding rules are satisfied.", "final_selfies": "[O][O][=C][=C][C][=C]"}
``` 

  

键合检验中都没有进行修订, 全部都是satisfied, 因为微调中并没有给 "需要修订"的案例

比起没有COT的时候, 发生了效果提升, 认为是COT过程触发了模型的某种思考 (比如模型进行了"谨慎对待")

  

# 对于"双键/三键个数错误", 尝试明确给出数量进行修正

提示词中明确指定原子数, 使用CoT流程, 明确指定双键/三键数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: 
     1. Correct: 72.00%, Predict selfies valid: 97.33%, Error ring: 0.00%, Error branch: 0.00%, Error atom_count: 6.67%, Error atom_seq: 8.00%, Error bond_count: 16.00%, Error bond_seq: 46.67%

     2. Correct: 72.00%, Predict selfies valid: 89.33%, error_type

NO 72.000000

error_atom_seq 9.333333

error_other 5.333333

error_atom_count 5.333333

error_bond_seq 5.333333

error_bond_count 2.666667

  4. 评估长度5: 
     1. Correct: 64.00%, Predict selfies valid: 91.00%, Error ring: 5.00%, Error branch: 3.00%, Error atom_count: 16.00%, Error atom_seq: 23.00%, Error bond_count: 6.00%, Error bond_seq: 52.00%

     2. Correct: 63.00%, Predict selfies valid: 90.00%, error_type

NO 62.0

error_atom_seq 10.0

error_bond_seq 7.0

error_atom_count 7.0

error_ring 5.0

error_bond_count 4.0

error_branch 3.0

error_other 2.0

  5. 评估长度6: 
     1. Correct: 49.00%, Predict selfies valid: 85.00%, Error ring: 9.00%, Error branch: 5.00%, Error atom_count: 24.00%, Error atom_seq: 34.00%, Error bond_count: 14.00%, Error bond_seq: 60.00%

     2. Correct: 43.00%, Predict selfies valid: 88.00%, error_type

NO 43.0

error_atom_seq 17.0

error_atom_count 12.0

error_bond_count 10.0

error_ring 9.0

error_other 4.0

error_bond_seq 3.0

error_branch 2.0

  

对比基线(epoch=1): 长度4的预测率下降9% (并不稳定重现), 在长度6上 error_bond_count 反而增加6个. 

看上去: 提示词中增加的键数信息并没有什么用

  

提示词中明确指定原子数, 使用CoT流程, 明确指定双键/三键数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  3. 评估长度4: 
     1. ~~Correct: 70.67%, Predict selfies valid: 94.67%, Error ring: 0.00%, Error branch: 0.00%, Error atom_count: 2.67%, Error atom_seq: 4.00%, Error bond_count: 14.67%, Error bond_seq: 44.00%~~

     2. Correct: 84.00%, Predict selfies valid: 97.33%, error_type

NO 84.000000

error_bond_count 6.666667

error_atom_seq 5.333333

error_bond_seq 1.333333

error_other 1.333333

error_atom_count 1.333333

  4. 评估长度5: 
     1. ~~Correct: 63.00%, Predict selfies valid: 95.00%, Error ring: 5.00%, Error branch: 3.00%, Error atom_count: 16.00%, Error atom_seq: 21.00%, Error bond_count: 11.00%, Error bond_seq: 53.00%~~

     2. Correct: 70.00%, Predict selfies valid: 98.00%, error_type

NO 70.0

error_bond_count 11.0

error_ring 5.0

error_atom_count 5.0

error_bond_seq 4.0

error_branch 3.0

error_atom_seq 1.0

error_other 1.0

  5. 评估长度6: 
     1. ~~Correct: 47.00%, Predict selfies valid: 96.00%, Error ring: 9.00%, Error branch: 5.00%, Error atom_count: 25.00%, Error atom_seq: 40.00%, Error bond_count: 11.00%, Error bond_seq: 55.00%~~

     2. Correct: 55.00%, Predict selfies valid: 95.00%, error_type

NO 55.0

error_atom_count 12.0

error_ring 9.0

error_atom_seq 8.0

error_bond_seq 8.0

error_bond_count 6.0

error_branch 2.0

对比基线(epoch=2): 长度4的预测率上升9% (并不稳定重现), 在长度4上 error_bond_count 降低10个, 在其他长度上没有太大变化. 长度5/6的准确率下降5-8%

看上去: 提示词中增加的键数信息在长度5/6上没有作用

提示词中明确指定原子数, 使用CoT流程, 明确指定双键/三键数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  3. 评估长度4: 
     1. Correct: 89.33%, Predict selfies valid: 100.00%, Error ring: 0.00%, Error branch: 0.00%, Error atom_count: 4.00%, Error atom_seq: 4.00%, Error bond_count: 6.67%, Error bond_seq: 38.67%

     2. Correct: 90.67%, Predict selfies valid: 98.67%, error_type

NO 90.666667

error_atom_count 4.000000

error_atom_seq 2.666667

error_bond_seq 1.333333

error_bond_count 1.333333

  4. 评估长度5: 
     1. Correct: 81.00%, Predict selfies valid: 95.00%, Error ring: 5.00%, Error branch: 3.00%, Error atom_count: 10.00%, Error atom_seq: 10.00%, Error bond_count: 4.00%, Error bond_seq: 39.00%

     2. Correct: 77.00%, Predict selfies valid: 94.00%, error_type

NO 77.0

error_atom_count 6.0

error_ring 5.0

error_bond_seq 5.0

error_branch 3.0

error_bond_count 3.0

error_other 1.0

  5. 评估长度6: 
     1. Correct: 64.00%, Predict selfies valid: 94.00%, Error ring: 9.00%, Error branch: 5.00%, Error atom_count: 21.00%, Error atom_seq: 24.00%, Error bond_count: 6.00%, Error bond_seq: 52.00%

     2. Correct: 63.00%, Predict selfies valid: 91.00%, error_type

NO 62.0

error_atom_count 14.0

error_ring 9.0

error_bond_count 5.0

error_bond_seq 5.0

error_atom_seq 2.0

error_branch 2.0

error_other 1.0

对比基线(epoch=4): 长度5的准确率下降6%, 长度4的error_bond_count下降4, 其他未有变化. 

看上去: 提示词中增加的键数信息在长度5/6上没有作用

提示词中明确指定原子数, 使用CoT流程, 明确指定双键/三键数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+6
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+6
  3. 评估长度4: 
     1. Correct: 90.67%, Predict selfies valid: 100.00%, Error ring: 0.00%, Error branch: 0.00%, Error atom_count: 2.67%, Error atom_seq: 4.00%, Error bond_count: 5.33%, Error bond_seq: 38.67%

     2. Correct: 93.33%, Predict selfies valid: 98.67%, error_type

NO 93.333333

error_atom_seq 2.666667

error_atom_count 1.333333

error_bond_seq 1.333333

error_bond_count 1.333333

  4. 评估长度5: 
     1. Correct: 79.00%, Predict selfies valid: 95.00%, Error ring: 5.00%, Error branch: 3.00%, Error atom_count: 10.00%, Error atom_seq: 12.00%, Error bond_count: 4.00%, Error bond_seq: 46.00%

     2. Correct: 85.00%, Predict selfies valid: 98.00%, error_type

NO 85.0

error_ring 5.0

error_atom_count 3.0

error_branch 3.0

error_bond_count 3.0

error_bond_seq 1.0

  5. 评估长度6: 
     1. Correct: 63.00%, Predict selfies valid: 95.00%, Error ring: 9.00%, Error branch: 5.00%, Error atom_count: 26.00%, Error atom_seq: 29.00%, Error bond_count: 10.00%, Error bond_seq: 53.00%

     2. Correct: 64.00%, Predict selfies valid: 95.00%, error_type

NO 64.0

error_atom_seq 9.0

error_ring 9.0

error_atom_count 9.0

error_bond_seq 3.0

error_other 2.0

error_branch 2.0

error_bond_count 2.0

对比基线, 没有明显提升

结论: 在提示词中增加对键数的提示, 并没有效果

# 对于"error_atom_count", 只改进COT流程

两种方法: 

  1. 模仿对原子数的提示, 给出各种原子的分项统计 (但在键数的理解上, 这种提示并没有作用)
  2. 改进COT流程, 增加对原子数的反思环节

(结论: 尝试过方法1后, 效果足够好, 并且错误已经转移到其他类型, 暂不尝试方案2)

### 先尝试第一种方式

生成脚本: [loop_selfies_and_image.generate_chain.CNO_count.cot.ipynb](/assets/01KJBZQ6DWHS6EB67JVEPB2H98/loop_selfies_and_image.generate_chain.CNO_count.cot.ipynb)

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: 
     1. Correct: 77.33%, Predict selfies valid: 96.00%, error_type

NO 77.333333

error_atom_seq 9.333333

error_bond_count 9.333333

error_bond_seq 2.666667

error_atom_count 1.333333

  4. 评估长度5: 
     1. Correct: 66.00%, Predict selfies valid: 93.00%, error_type

NO 65.0

error_atom_seq 9.0

error_bond_count 8.0

error_atom_count 6.0

error_ring 5.0

error_other 3.0

error_branch 3.0

error_bond_seq 1.0

  5. 评估长度6: 
     1. Correct: 46.00%, Predict selfies valid: 84.00%, error_type

NO 45.0

error_atom_seq 23.0

error_ring 9.0

error_bond_count 9.0

error_atom_count 7.0

error_bond_seq 5.0

error_branch 2.0

  

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  3. 评估长度4: 
     1. Correct: 85.33%, Predict selfies valid: 94.67%, error_type

NO 85.333333

error_bond_count 8.000000

error_atom_seq 5.333333

1.333333

  4. 评估长度5: 
     1. Correct: 71.00%, Predict selfies valid: 98.00%, error_type

NO 71.0

error_bond_count 9.0

error_atom_seq 8.0

error_ring 5.0

error_branch 3.0

error_bond_seq 2.0

error_other 2.0

  5. 评估长度6: 
     1. Correct: 58.00%, Predict selfies valid: 95.00%, error_type

NO 57.0

error_atom_seq 17.0

error_ring 9.0

error_bond_count 9.0

error_atom_count 3.0

error_bond_seq 3.0

error_branch 2.0

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  3. 评估长度4: 
     1. Correct: 89.33%, Predict selfies valid: 98.67%, error_type

NO 89.333333

error_bond_count 9.333333

error_other 1.333333

  4. 评估长度5: 
     1. Correct: 81.00%, Predict selfies valid: 96.00%, error_type

NO 81.0

error_ring 5.0

error_bond_count 4.0

error_atom_seq 3.0

error_branch 3.0

error_bond_seq 2.0

error_other 1.0

error_atom_count 1.0

  5. 评估长度6: 
     1. Correct: 65.00%, Predict selfies valid: 92.00%, error_type

NO 65.0

error_ring 9.0

error_atom_seq 8.0

error_bond_count 7.0

error_bond_seq 5.0

error_other 3.0

error_branch 2.0

error_atom_count 1.0

  

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+6
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+6
  3. 评估长度4: 
     1. Correct: 89.33%, Predict selfies valid: 98.67%, error_type

NO 89.333333

error_bond_count 8.000000

error_atom_seq 2.666667

  4. 评估长度5: 
     1. Correct: 84.00%, Predict selfies valid: 98.00%, error_type

NO 83.0

error_bond_count 8.0

error_ring 4.0

error_branch 3.0

error_atom_seq 1.0

1.0

  5. 评估长度6: 
     1. Correct: 72.00%, Predict selfies valid: 97.00%, error_type

NO 71.0

error_ring 9.0

error_atom_seq 7.0

error_bond_count 6.0

error_bond_seq 3.0

error_atom_count 2.0

error_branch 2.0

  

与基线对比, 在长度4/5/6上, error_atom_count明显下降, 但整体准确率没有提升, 其错误会向error_atom_seq和error_bond_count转移, 其中需要评估转移的程度

观察epoch1/2/4/6的总和: 

  - 基线: error_atom_seq: 56个, error_atom_count: 103个, error_bond_count: 82个, error_bond_count: 28个, error_ring: 55个
  - 转移后: error_atom_seq: 93个, error_atom_count: 21个, error_bond_count: 95个, error_bond_count: 23个, error_ring: 55个

看到原子数错误明显下降, 错误大部分(37个)转移到error_atom_seq, 少量(13个)转移到error_bond_count

# 对于"error_atom_seq", 尝试改进COT流程, 进行修正

参考 error_bond_count 的教训 (仅改进COT流程就可以大幅改进效果, 而增加训练数据的效果不大), 先尝试修改COT流程

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, COT流程中要求对各原子的分项数量进行思考

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  3. 评估长度4: 
     1. Correct: 84.00%, Predict selfies valid: 98.67%, error_type

NO 84.000000

error_bond_count 10.666667

error_atom_seq 5.333333

  4. 评估长度5: 
     1. Correct: 68.00%, Predict selfies valid: 95.00%, error_type

NO 67.0

error_bond_count 10.0

error_atom_seq 9.0

error_ring 5.0

error_atom_count 4.0

error_branch 3.0

error_bond_seq 2.0

  5. 评估长度6: 
     1. Correct: 44.00%, Predict selfies valid: 91.00%, error_type

NO 43.0

error_atom_seq 20.0

error_bond_count 11.0

error_atom_count 10.0

error_ring 9.0

error_bond_seq 4.0

error_branch 2.0

error_other 1.0

  

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, COT流程中要求对各原子的顺序 进行思考

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+2
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+2
  3. 评估长度4: 
     1. Correct: 73.33%, Predict selfies valid: 98.67%, error_type

NO 73.333333

error_bond_count 12.000000

error_atom_seq 10.666667

error_atom_count 2.666667

error_bond_seq 1.333333

  4. 评估长度5: 
     1. Correct: 68.00%, Predict selfies valid: 97.00%, error_type

NO 67.0

error_bond_count 11.0

error_atom_seq 8.0

error_ring 5.0

error_atom_count 4.0

error_branch 3.0

error_bond_seq 2.0

  5. 评估长度6: 
     1. Correct: 50.00%, Predict selfies valid: 94.00%, error_type

NO 50.0

error_atom_seq 17.0

error_ring 9.0

error_bond_seq 9.0

error_atom_count 6.0

error_bond_count 6.0

error_branch 2.0

error_other 1.0

  

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, COT流程中要求对各原子的顺序 进行思考

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+4
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+4
  3. 评估长度4: 
     1. Correct: 88.00%, Predict selfies valid: 98.67%, error_type

NO 88.000000

error_bond_count 10.666667

error_atom_seq 1.333333

  4. 评估长度5: 
     1. Correct: 75.00%, Predict selfies valid: 98.00%, error_type

NO 75.0

error_bond_count 12.0

error_ring 4.0

error_branch 3.0

error_atom_seq 3.0

error_atom_count 2.0

1.0

  5. 评估长度6: 
     1. Correct: 61.00%, Predict selfies valid: 91.00%, error_type

NO 61.0

error_bond_count 11.0

error_ring 9.0

error_bond_seq 6.0

error_atom_count 6.0

error_atom_seq 5.0

error_branch 2.0

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, COT流程中要求对各原子的顺序 进行思考

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+6
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+6
  3. 评估长度4: 
     1. Correct: 90.67%, Predict selfies valid: 100.00%, error_type

NO 90.666667

error_bond_count 6.666667

error_atom_seq 2.666667

  4. 评估长度5: 
     1. Correct: 82.00%, Predict selfies valid: 94.00%, error_type

NO 82.0

error_ring 5.0

error_atom_seq 4.0

error_branch 3.0

error_bond_count 3.0

error_bond_seq 2.0

error_other 1.0

  5. 评估长度6: 
     1. Correct: 65.00%, Predict selfies valid: 92.00%, error_type

NO 63.0

error_atom_seq 11.0

error_ring 9.0

error_atom_count 6.0

error_bond_count 5.0

error_bond_seq 4.0

error_branch 2.0

  

对于error_atom_seq, 与COT基线相比, 均未见明显改善. 也就是说符合之前的判断: 

比起没有COT的时候, 发生了效果提升, 认为是COT过程触发了模型的某种思考风格 (比如模型进行了"谨慎对待"). 

但并不是模型进行了按步骤的思考 (因为微调训练数据中没有给相关案例). 所以增加模型在思考过程中的条目, 并不会改善最终效果.

  

# 对于"error_atom_seq", 改进COT流程, 并增加相关COT的训练数据

调整数据生成脚本: [loop_selfies_and_image.generate_chain.CNO_count.correct_atom_seq.cot.ipynb](/assets/01KJBZQ6DWHS6EB67JVEPB2H98/loop_selfies_and_image.generate_chain.CNO_count.correct_atom_seq.cot.ipynb) , 其中:

  - 每个数据生成一个正确数据, 以及一个修正数据 (将其中某个原子更换位置, 要求模型调整对)

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 增加了对原子位置的修正数据

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (增加了一倍修正数据, 带数据增强, 取全量), epoch+1
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (增加了一倍修正数据, 带数据增强, 取全量), epoch+1
  3. 评估长度4: 
     1. Correct: 46.67%, Predict selfies valid: 77.33%, error_type

NO 45.333333

error_atom_seq 38.666667

error_bond_seq 6.666667

error_bond_count 6.666667

error_other 2.666667

  4. 评估长度5: 
     1. Correct: 30.00%, Predict selfies valid: 78.00%, error_type

error_atom_seq 47.0

NO 30.0

error_bond_seq 9.0

error_ring 5.0

error_bond_count 3.0

error_branch 3.0

error_other 2.0

error_atom_count 1.0

  5. 评估长度6: 
     1. Correct: 13.00%, Predict selfies valid: 71.00%, error_type

error_atom_seq 55.0

NO 13.0

error_bond_seq 12.0

error_ring 9.0

error_atom_count 6.0

error_branch 2.0

error_bond_count 2.0

error_other 1.0

  

与基线相比, 正确性全面降低, 分析长度4的评估结果: [evaluate_1.4.log](/assets/01KJBZQ6DWHS6EB67JVEPB2H98/evaluate_1.4.log), 有部分数据是从正确被"修正"成错误的: 

```
19  /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/187.png                    {"initial_selfies": "[O][N][O][C]", "validation": "Error detected: The atom '[O]' at position 3 seems out of place.", "final_selfies": "[O][O][N][C]"}             OONC     {"single_bonds": 3, "double_bonds": 0, "triple_bonds": 0, "ring_count": 0, "C_count": 1, "O_count": 2, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[O][N][O][C]"}      ONOC    0.285714                    True        False          False              False             True              False            False    error_atom_seq
 
9   /opt/huangyan/LLaMA-Factory/data/loop_selfies/length_4/367.png               {"initial_selfies": "[O][C][=C][#N]", "validation": "Error detected: The atom '[=C]' at position 3 seems out of place.", "final_selfies": "[O][C][#N][=C]"}             OC#N   {"single_bonds": 1, "double_bonds": 1, "triple_bonds": 1, "ring_count": 0, "C_count": 2, "O_count": 1, "N_count": 1, "branch_count": 0, "atom_count": 4, "selfies": "[N][#C][C][=O]"}    N#CC=O    0.250000                   False        False          False              False             True              False             True    error_atom_seq
``` 

  

正确数据和修正数据都放在同一个文件中, 那么长度1-3的数据, 应当等比增大(取200个):

  

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 增加了对原子位置的修正数据

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取200个), 长度4数据 (增加了一倍修正数据, 带数据增强, 取全量), epoch+1
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取200个), 长度5数据 (增加了一倍修正数据, 带数据增强, 取全量), epoch+1
  3. 评估长度4: 
     1. Correct: 54.67%, Predict selfies valid: 89.33%, error_type

NO 54.666667

error_atom_seq 22.666667

error_bond_seq 9.333333

error_bond_count 6.666667

error_other 5.333333

error_atom_count 1.333333

  4. 评估长度5: 
     1. Correct: 42.00%, Predict selfies valid: 86.00%, error_type

NO 42.0

error_atom_seq 37.0

error_bond_count 5.0

error_bond_seq 5.0

error_ring 5.0

error_branch 3.0

error_other 2.0

error_atom_count 1.0

  5. 评估长度6: 
     1. Correct: 20.00%, Predict selfies valid: 82.00%, error_type

error_atom_seq 46.0

NO 20.0

error_ring 9.0

error_bond_seq 8.0

error_bond_count 8.0

error_atom_count 7.0

error_branch 2.0

  

比起 长度1-3数据取100个 的情况, error_atom_seq降低, 整体准确率上升. 

也就是说: 可能是 修正数据比例过高, 模型无法认知到"正确的case"

将同样的情况, epoch=2/4的数据跑完作为参考: 

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 增加了对原子位置的修正数据

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取200个), 长度4数据 (增加了一倍修正数据, 带数据增强, 取全量), epoch+2
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取200个), 长度5数据 (增加了一倍修正数据, 带数据增强, 取全量), epoch+2
  3. 评估长度4: 
     1. Correct: 74.67%, Predict selfies valid: 93.33%, error_type

NO 74.666667

error_atom_seq 9.333333

error_bond_count 6.666667

error_bond_seq 6.666667

error_atom_count 2.666667

  4. 评估长度5: 
     1. Correct: 63.00%, Predict selfies valid: 86.00%, error_type

NO 63.0

error_atom_seq 15.0

error_bond_seq 6.0

error_bond_count 5.0

error_ring 5.0

error_branch 3.0

error_other 2.0

error_atom_count 1.0

  5. 评估长度6: 
     1. Correct: 36.00%, Predict selfies valid: 84.00%, error_type

error_atom_seq 37.0

NO 36.0

error_ring 9.0

error_bond_seq 6.0

error_atom_count 5.0

error_bond_count 3.0

error_other 2.0

error_branch 2.0

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 增加了对原子位置的修正数据

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取200个), 长度4数据 (增加了一倍修正数据, 带数据增强, 取全量), epoch+4
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取200个), 长度5数据 (增加了一倍修正数据, 带数据增强, 取全量), epoch+4
  3. 评估长度4: 
     1. Correct: 80.00%, Predict selfies valid: 94.67%, error_type

NO 80.000000

error_atom_seq 10.666667

error_bond_seq 2.666667

error_atom_count 2.666667

error_other 2.666667

error_bond_count 1.333333

  4. 评估长度5: 
     1. Correct: 67.00%, Predict selfies valid: 94.00%, error_type

NO 67.0

error_atom_seq 13.0

error_bond_seq 6.0

error_bond_count 6.0

error_ring 4.0

error_branch 3.0

1.0

  5. 评估长度6: 
     1. Correct: 46.00%, Predict selfies valid: 81.00%, error_type

NO 46.0

error_atom_seq 24.0

error_bond_seq 10.0

error_ring 9.0

error_atom_count 6.0

error_bond_count 3.0

error_branch 2.0

  

  

使用多步训练:

  1. 开始训练, 最终任务(描述如上), 仅包括正向案例, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  2. 继续训练, 最终任务(描述如上), 包括正向案例和反向案例, 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (带数据增强, 取全量), epoch+1
  3. 继续训练, 最终任务(描述如上), 仅包括正向案例, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  4. 继续训练, 最终任务(描述如上), 包括正向案例和反向案例, 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (带数据增强, 取全量), epoch+1
  5. 评估长度4: 
     1. Correct: 46.67%, Predict selfies valid: 84.00%, error_type

NO 46.666667

error_atom_seq 26.666667

error_bond_seq 12.000000

error_bond_count 12.000000

error_other 2.666667

  6. 评估长度5: 
     1. Correct: 41.00%, Predict selfies valid: 91.00%, error_type

NO 41.0

error_atom_seq 33.0

error_bond_count 8.0

error_bond_seq 5.0

error_ring 5.0

error_atom_count 4.0

error_branch 3.0

error_other 1.0

  7. 评估长度6: 
     1. Correct: 12.00%, Predict selfies valid: 76.00%, error_type

error_atom_seq 59.0

NO 12.0

error_ring 9.0

error_bond_seq 8.0

error_atom_count 6.0

error_bond_count 3.0

error_branch 2.0

1.0

降低修正数据的量

提示词中明确指定原子数, 使用CoT流程, 明确指定CNO的数量, 增加了对原子位置的修正数据

  1. 开始训练, 最终任务(描述如上), 长度1-3数据 (带数据增强, 最多取100个), 长度4数据 (增加了50个修正数据, 带数据增强, 取全量), epoch+1
  2. 继续训练, 最终任务(描述如上), 长度1-4数据 (带数据增强, 最多取100个), 长度5数据 (增加了50个修正数据, 带数据增强, 取全量), epoch+1
  3. 评估长度4: 
     1. Correct: 62.67%, Predict selfies valid: 92.00%, error_type

NO 62.666667

error_atom_seq 20.000000

error_bond_count 10.666667

error_bond_seq 4.000000

error_atom_count 2.666667

  4. 评估长度5: 
     1. Correct: 51.00%, Predict selfies valid: 89.00%, error_type

NO 51.0

error_atom_seq 22.0

error_bond_count 9.0

error_ring 5.0

error_bond_seq 5.0

error_atom_count 3.0

error_branch 3.0

error_other 2.0

  5. 评估长度6: 
     1. Correct: 35.00%, Predict selfies valid: 87.00%, error_type

NO 34.0

error_atom_seq 27.0

error_bond_count 11.0

error_ring 9.0

error_atom_count 9.0

error_bond_seq 8.0

error_branch 2.0

比起基线, 性能整体下降, 但比起使用更多修正数据的情况, 性能整体回升.

看上去 修正数据 起到了相反的作用

  

  

# 下一步

还是考虑使用两种相似的COT流程, 而不是使用一个COT流程串起两种任务
