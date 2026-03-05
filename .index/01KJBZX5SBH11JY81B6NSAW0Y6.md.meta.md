---
note: 01KJBZX5SBH11JY81B6NSAW0Y6.md
title: 20250811 - 诊断GRPO训练过程的时间/内存消耗
indexed_at: 2026-03-05T11:52:39.820112+00:00
---

## 摘要
记录 GRPO 训练过程中的时间和内存消耗诊断方法，使用 SwanLab 和 patch_profiling 工具标记关键代码块进行性能监控。统一日志输出格式，便于分析 VLLM 负载情况和代码调用次数，定位训练过程中的性能瓶颈。

## 关键概念
- patch_profiling_context: 用于标记和监控 ms-swift 中关键代码块性能的工具
- SwanLab: 通过启动参数集成的训练过程监控和报告平台
- 日志格式统一化: 通过遍历所有 logger 的 handler 统一 formatter，便于后续日志分析

## 关联笔记
- 01KJBZZJ92R5GJ0VH1J4W3X93K.md: 后续 SQL 优化 GRPO 训练的数据生成过程改进
- 01KJBZY39GJE38CB7BYSTRWMBX.md: 引用本笔记结论进行内存和时间优化
- 01KJBZW4XHSH3DH775CDV24XT5.md: GRPO 训练中 train/loss 和 reward 的理论分析
