---
note: 01KJBZRSYVSHF6VCBP86KAQ3RD.md
title: 20250306 - 对分子式识别的微调任务经验的第一阶段 进行总结
indexed_at: 2026-03-05T11:32:31.561999+00:00
---

## 标签
分子式识别, 微调, Qwen2-VL, LoRA, 课程训练, SELFIES

## 摘要
总结分子式识别微调任务第一阶段（20241112-20250215）的经验，涵盖 LoRA 层数调整、SELFIES 表达式、loss 函数修改、数据选择等关键发现。指出评估集独立性、梯度贡献数据选择、课程训练参数配置等实践要点。

## 关键概念
- LoRA 微调: 减少可训练层数可减轻过拟合现象
- SELFIES 表达式: 比 SMILES 更易理解且鲁棒性更好的分子表示方法
- 梯度贡献指标: 用于选择最有特点的训练数据，但数据量仍是关键
- 课程训练: 递增式训练需手工计算 max_step 或使用 resume_from_checkpoint
- 多任务训练: 增加原子数预测等辅助任务，但整体正确率提升有限

## 关联笔记
- 01KJBZRRGYWH5WHAZ2TZKMP91F.md: 工作移交笔记，记录第一阶段结束和第二阶段开始
- 01KJBZR0H7TBJT1Y07GE1DHEBD.md: 课程学习和 resume_from_checkpoint 实践的来源笔记
- 01KJBZR66G465R50X6V0TXKB9K.md: 学习率对推理能力影响的深入分析
