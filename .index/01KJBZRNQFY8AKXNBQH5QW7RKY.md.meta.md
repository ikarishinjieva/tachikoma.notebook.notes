---
note: 01KJBZRNQFY8AKXNBQH5QW7RKY.md
title: 20250215 - 使用ms-swift对Qwen2-VL进行微调 [25] - 扩展长度9
indexed_at: 2026-03-05T11:30:17.295434+00:00
---

## 摘要
记录 Qwen2-VL 微调实验中从长度 8 扩展到长度 9 的训练结果，基础原子和 branch 分子表现尚可但 ring 分子识别率仅 25-35%。通过详细分析错误案例，发现主要问题包括原子位置对调、镜像变换、branch 长度错误和 Ring 操作符误用等模式。

## 关键概念
- SELFIES: 分子结构的字符串表示法，比 SMILES 更鲁棒且易于理解
- 课程训练: 按难度递增（长度 7→8→9）的渐进式训练策略
- Branch 操作符: SELFIES 中表示分子分支结构的标记
- 镜像变换: 分子结构识别错误的一种类型，预测结果呈镜像对称

## 关联笔记
- 01KJBZRGRZ1WMJ4QQSBPPYVYJN.md: 直接前置基线，记录长度 7-8 的收敛研究
- 01KJBZQQVCZ5ZSZZ4W1F9974D.md: 后续第 26 篇，基于本笔记经验继续优化模型学习策略
- 01KJBZRRGYWH5WHAZ2TZKMP91F.md: 第 27 篇工作移交，总结本笔记中长度扩展到 9 的经验教训
