---
note: 01KJBZKX971VTKSDKYRNX3ZV7N.md
title: 20241121 - 使用ms-swift对Qwen2-VL进行微调 [7] - 用规范数据减少过拟合
indexed_at: 2026-03-05T11:01:23.753863+00:00
---

## 摘要
使用 ms-swift 对 Qwen2-VL-7B 进行 LoRA 微调，通过递增长度（1-4）的 SELFIES 规范数据训练模型识别分子结构图。训练集出现过拟合，评估发现模型对"三键 + 双键"结构识别困难，但部分 SELFIES 表达式不一致的样例实际化学结构相同。

## 关键概念
- SELFIES: 自参考嵌入式字符串表示法，用于表示分子结构，比 SMILES 更鲁棒
- LoRA: 低秩适配器，一种参数高效的微调方法
- 过拟合: 模型在训练集上表现好但在新数据上泛化能力差的现象
- ms-swift: 魔搭社区的多模态模型微调框架
- 课程学习: 从简单到复杂逐步增加训练数据难度的策略

## 关联笔记
- 01KJBZKTQRNA64HW30T7QG691S.md: 前序实验 [6]，探索增强 SELFIES 效果的方法
- 01KJBZM0EKNHYYJ7Q27SMJ2WV6.md: 后续实验 [8]，继续用规范数据减少过拟合的第二次尝试
- 01KJBZRSYVSHF6VCBP86KAQ3RD.md: 阶段总结，引用本笔记作为关键经验来源
