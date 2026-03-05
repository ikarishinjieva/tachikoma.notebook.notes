---
note: 01KJBZRF4FT28F7NFSZD176TXQ.md
title: 20250211 - 阅读论文*: V-STaR: Training Verifiers for Self-Taught Reasoners
indexed_at: 2026-03-05T11:28:00.663807+00:00
---

## 摘要
V-STaR 在 STaR 基础上引入验证器，利用 DPO 训练验证器来区分正确与错误答案。相比 STaR 直接丢弃错误答案，V-STaR 通过验证器更有效地利用训练数据，实现性能提升。

## 关键概念
- V-STaR: 通过引入验证器并利用错误答案训练，改进 STaR 的方法
- STaR: 利用模型生成的正确答案迭代训练模型，无独立验证器
- 验证器 (Verifier): 独立模型，用于判断答案正误，通过 DPO 训练
- DPO: 直接偏好优化，用于训练验证器区分正确与错误答案
- LoRA: 参数高效微调方法，V-STaR 使用此方式更新参数

## 关联笔记
- 01KJBZR2DKSYYSHN1HFGATWV4H.md: STaR 论文笔记，V-STaR 的直接前身方法
- 01KJBZRFKPHCGB3ETQ4MS9CVH0.md: CoRe 论文笔记，同样使用验证器和类 STaR 迭代训练方法
