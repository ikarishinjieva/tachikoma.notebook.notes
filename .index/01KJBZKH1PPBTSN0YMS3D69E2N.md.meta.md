---
note: 01KJBZKH1PPBTSN0YMS3D69E2N.md
title: 20241115 - ms-swift是如何计算acc的
indexed_at: 2026-03-05T10:59:33.851928+00:00
---

## 摘要
分析 ms-swift 框架中 compute_acc_metrics 函数的实现逻辑，解释 token 级别和句子级别两种准确率计算策略的差异。函数处理编码器 - 解码器结构、padding tokens 掩码，并对比 loss 与 acc 的评估目标差异。

## 关键概念
- compute_acc_metrics: 计算生成式模型准确率的函数，支持 token 和句子两种策略
- token 级别准确率: 计算所有有效 token 预测正确的平均值
- 句子级别准确率: 句子中所有有效 token 完全匹配才算该句子正确
- 掩码 (masks): 标识有效 token 位置，排除 padding tokens(-100) 参与计算

## 关联笔记
- 01KJBZJMH94H8E9GBJBFTFCW58.md: 讨论 transformers 的 evaluate 机制，解释 token 级别和序列级别评估的区别
- 01KJBZJSM5YEKX18QPBPFMAYNX.md: 分析模型评估过拟合原因，涉及评估指标的使用
