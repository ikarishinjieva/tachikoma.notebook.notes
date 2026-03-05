---
note: 01KJBZJYMDWQXWHPAVTRVWV50H.md
title: 20241114 - 使用ms-swift对Qwen2-VL进行微调 [4] - 修改loss
indexed_at: 2026-03-05T10:59:06.118390+00:00
---

## 摘要
记录使用 loss_scale 配置增强 SMILES 表达式权重的实验，遇到 mask 形状与 labels 长度不匹配的 IndexError。绕过 bug 后训练结果显示 loss 整体抬升，权重调整未能驱动模型找到更优解。

## 关键概念
- loss_scale: 通过配置文件为特定 token 增加 loss 权重，引导模型关注重点内容
- IndexError: loss_scale 计算的数组长度与模板增长后的 labels 长度不匹配导致
- LoRA: 使用低秩适配器对 Qwen2-VL-7B-Instruct 进行参数高效微调

## 关联笔记
- 01KJBZJWFDR591GB209N8PKJ1F.md: 同一系列 [3]，前一篇尝试使用更多数据但效果无改善
- 01KJBZKJR3KRVFZRXR50RG57YM.md: 同一系列 [5]，后一篇总结本篇结论为增强 loss 权重对效果无改善
- 01KJBZRSYVSHF6VCBP86KAQ3RD.md: 第一阶段经验总结，引用本篇说明调整 loss 无法驱动模型找到更准确解
