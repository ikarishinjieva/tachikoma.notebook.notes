---
note: 01KJBZY39GJE38CB7BYSTRWMBX.md
title: 20250813 - 继续进行SQL优化的GRPO训练
indexed_at: 2026-03-05T11:53:19.593320+00:00
---

## 摘要
分析了 GRPO 训练中 completion 长度与内存/时间消耗的关系，确定可通过缩小 model length 参数换取训练效率。修复尝试 11 中训练/评估奖励崩溃问题，包括多列模式支持、as 匹配逻辑升级、多种删除判断格式支持。尝试 12 中 eval F1 reward 异常最终定位到 max_length 参数配置不合理。

## 关键概念
- GRPO: 用于 SQL 优化的强化学习训练方法
- Reward 函数: 评估模型输出的奖励机制，支持多列模式和多种删除判断格式
- max_length: 控制模型 prompt 长度的关键参数，配置不当会导致评估异常
- max_completion_length: 控制模型完成部分的最大生成长度
- vllm_max_model_len: vLLM 推理引擎的模型长度上限

## 关联笔记
- 01KJBZW8ND6Q8NTEDX7B4WWACP.md: 前继笔记，记录尝试 11 的 SQL 优化 GRPO 训练方案及奖励崩溃问题
- 01KJBZX5SBH11JY81B6NSAW0Y6.md: 前继笔记，诊断 GRPO 训练过程的时间/内存消耗问题并给出优化建议
