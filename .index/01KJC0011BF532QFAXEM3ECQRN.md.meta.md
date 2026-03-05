---
note: 01KJC0011BF532QFAXEM3ECQRN.md
title: 20250911 - 阅读论文*: ON LARGE-BATCH TRAINING FOR DEEP LEARNING: GENERALIZATION GAP AND SHARP MINIMA
indexed_at: 2026-03-05T11:57:37.645981+00:00
---

## 摘要
该论文研究了大批次训练导致模型泛化性变差的现象。核心原因是大批次削减了批次内数据的噪声，使模型容易陷入次优解的陡峭吸引盆而无法逃逸；小批次训练则因噪声随机性高，更容易找到平坦且泛化性更好的最优解。

## 关键概念
- 大批次训练: 使用较大 batch size 进行参数更新，会削减批次内数据的噪声
- 吸引盆 (Attraction Basin): 损失景观中局部最优解周围的区域，模型训练容易陷入其中
- 损失景观 (Loss Landscape): 将模型参数空间映射到损失值的几何表示，平坦盆地对应更好泛化性
- 梯度噪声: 小批次训练引入的随机性，有助于从次优解中逃逸

## 关联笔记
- 01KJBZX5SBH11JY81B6NSAW0Y6.md: 提到了减小批次大小作为训练优化的缓解措施
- 01KJBZN5YYTD5QQ8Z1D621Z88K.md: 讨论了梯度大小与泛化性的关系，梯度更大导致泛化性升高
- 01KJBZKX971VTKSDKYRNX3ZV7N.md: 探讨了学习率调整对泛化性的影响，与训练策略优化相关
