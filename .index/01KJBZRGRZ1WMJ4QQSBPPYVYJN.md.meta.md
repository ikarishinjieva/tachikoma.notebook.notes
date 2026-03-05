---
note: 01KJBZRGRZ1WMJ4QQSBPPYVYJN.md
title: 20250212 - 使用ms-swift对Qwen2-VL进行微调 [24] - 研究训练数据的收敛
indexed_at: 2026-03-05T11:29:06.744248+00:00
---

## 摘要
研究 Qwen2-VL 微调中训练数据收敛问题，基于三阶段课程学习基线（epoch=3/6/9）分析 branch 分子识别错误。通过错误分析发现 branch 位置错误、原子调序、branch 长度错误等典型问题，并尝试多种数据组合策略改善收敛。

## 关键概念
- 课程学习：分三阶段递减学习率（1e-4→5e-5→3e-5）逐步训练的策略
- branch 分子：含分支结构的分子表示，识别难度高于直链和 ring 结构
- 数据收敛：训练数据在增加 epoch 时效果趋于稳定的状态
- 原子调序错误：branch 内外原子顺序混淆导致的识别错误

## 关联笔记
- 01KJBZRBSBD6C78ZNC9ERG9QB3.md: 前序笔记，建立了三阶段课程学习基线
- 01KJBZRNQFY8AKXNBQH5QW7RKY.md: 后续笔记，将训练长度扩展到 9 并继续优化
- 01KJBZRQQVCZ5ZSZZ4W1F9974D.md: 系列后续，让模型学习"变化"的进一步研究
