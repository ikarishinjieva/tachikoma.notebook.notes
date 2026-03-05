---
note: 01KJBZKJR3KRVFZRXR50RG57YM.md
title: 20241115 - 使用ms-swift对Qwen2-VL进行微调 [5] - 解决过拟合状态
indexed_at: 2026-03-05T11:00:06.871027+00:00
---

## 摘要
记录 Qwen2-VL 微调第 5 次实验，通过建立 1000 数据基线确认过拟合出现时间点。验证数据增强和 SELFIES 表达式替换方案，发现简单图片增强无法改善过拟合（评估集与训练集同源导致虚假 loss 下降），SELFIES 因序列更长导致 loss 计算分母增大。

## 关键概念
- 过拟合检测: 通过 (eval-train)/eval 和 grad_norm*LR 指标判断过拟合出现时间
- 数据增强: 对训练图片进行变换增殖，但简单变换无法改善 Qwen2-VL 训练效果
- 评估集独立性: 评估集必须与训练集完全独立，否则会导致虚假的 loss 下降
- SELFIES 表达式: 替代 SMILES 的分子表示法，序列更长更鲁棒但影响 loss 计算

## 关联笔记
- 01KJBZJVK3NEV6BHJ6K77YKYC5.md: 前序实验 [2]，尝试用 SELFIES 替代 SMILES 表达式
- 01KJBZJWFDR591GB209N8PKJ1F.md: 前序实验 [3]，测试更多数据对效果的影响
- 01KJBZJYMDWQXWHPAVTRVWV50H.md: 前序实验 [4]，修改 loss 权重增强结论部分
