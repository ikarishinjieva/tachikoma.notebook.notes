---
note: 01KJBZMAPMJ1WPHS51T0ZMXHCE.md
title: 20241128 - 使用ms-swift对Qwen2-VL进行微调 [9] - 用梯度贡献来进行数据集的选择
indexed_at: 2026-03-05T11:02:34.195239+00:00
---

## 摘要
记录使用梯度贡献选择训练数据的实验，比较选取 300 个和 500 个梯度贡献最大的样本进行 LoRA 微调的效果。发现小数据集容易过拟合，train loss 收敛快但 eval loss 中途爆炸。

## 关键概念
- 梯度贡献选择: 遍历所有数据，选择梯度贡献最大的样本加入下一轮训练
- 过拟合诊断: train loss 收敛快但 eval loss 爆炸表明模型过拟合
- LoRA 微调: 使用低秩适配器对 Qwen2-VL-7B-Instruct 进行参数高效微调

## 关联笔记
- 01KJBZRSYVSHF6VCBP86KAQ3RD.md: 总结整个微调任务经验，引用了本笔记的梯度贡献选择方法
- 01KJBZRNQFY8AKXNBQH5QW7RKY.md: 后续微调实验，继续探索课程训练和长度扩展
