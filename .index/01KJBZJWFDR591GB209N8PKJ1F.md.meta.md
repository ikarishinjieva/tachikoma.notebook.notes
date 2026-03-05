---
note: 01KJBZJWFDR591GB209N8PKJ1F.md
title: 20241114 - 使用ms-swift对Qwen2-VL进行微调 [3] - 使用更多数据
indexed_at: 2026-03-05T10:58:36.364933+00:00
---

## 标签
Qwen2-VL, ms-swift, LoRA 微调, 过拟合, 数据量实验, 训练日志

## 摘要
记录 Qwen2-VL 微调实验中调整数据量（6000→8000→10000）和校验集比例（0.05）对训练效果的影响。发现增加数据量对 train/eval loss 改善无明显作用，过拟合点从 step 900 提前至 step 600。

## 关键概念
- 过拟合 (Overfitting): 训练 loss 持续下降但验证 loss 不再改善的现象
- dataset_test_ratio: 校验集占总数据的比例，调高可更准确评估泛化能力
- LoRA: 低秩适配器微调方法，仅训练少量参数
- 学习率周期: LR 变化周期拉长导致 loss 下降周期同步拉长

## 关联笔记
- 01KJBZJT9YFKP1XPAKC0PHG4GA.md: 系列 [1] 基线实验，本笔记从其 v0 版本出发进行调整
- 01KJBZJVK3NEV6BHJ6K77YKYC5.md: 系列 [2] 同一天实验，尝试用 SELFIES 替代 SMILES 表示
