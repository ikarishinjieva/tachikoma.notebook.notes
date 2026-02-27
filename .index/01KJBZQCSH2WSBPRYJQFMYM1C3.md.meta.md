---
note: 01KJBZQCSH2WSBPRYJQFMYM1C3.md
title: 20250111 - 使用ms-swift对Qwen2-VL进行微调 [18] - 使用相似COT流程进行两种互补任务
indexed_at: 2026-02-27T11:26:54.651767+00:00
---

用户要求我分析这篇关于使用 ms-swift 对 Qwen2-VL 进行微调的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **主题**: 使用 ms-swift 框架对 Qwen2-VL 视觉语言模型进行微调，专注于化学分子识别任务（SMILES/SELFIES 表示）

2. **核心内容**:
   - 对预测错误进行分类，主要关注 atom_seq 错误
   - 使用相似 COT（Chain of Thought）流程进行两种互补任务：识别任务和修正任务
   - 比较一阶段评估和两阶段评估（先识别 selfies，再修正）
   - 多个 epoch 的训练结果和评估数据

3. **关键概念**:
   - ms-swift: 模型微调框架
   - Qwen2-VL: 视觉语言模型
   - COT (Chain of Thought): 思维链流程
   - SELFIES: 分子表示方法
   - atom_seq 错误: 原子序列错误

4. **关联笔记**: 查看提供的笔记列表，这些都是关于数据库、MySQL、Clickhouse、系统调试等内容的，与这篇关于 AI 模型微调和化学分子识别的笔记没有明显关联。

现在生成索引：## 标签
Qwen2-VL, ms-swift, 模型微调, COT 流程, 分子识别, SELFIES

## 摘要
记录使用 ms-swift 框架对 Qwen2-VL 进行微调的实验过程，采用相似 COT 流程同时训练识别任务和修正任务。通过多轮 epoch 训练，对比一阶段和两阶段评估方案在分子结构预测上的效果，重点关注降低 atom_seq 错误率。

## 关键概念
- COT 流程: 思维链（Chain of Thought）方法，引导模型逐步推理
- SELFIES: 一种分子结构的字符串表示方法，用于化学分子识别
- 两阶段评估: 先预测 SELFIES 再进行修正的分步评估方案
- atom_seq 错误: 原子序列预测错误的类型分类

## 关联笔记
无
