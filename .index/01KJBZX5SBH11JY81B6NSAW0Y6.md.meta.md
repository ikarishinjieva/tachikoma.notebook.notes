---
note: 01KJBZX5SBH11JY81B6NSAW0Y6.md
title: 20250811 - 诊断GRPO训练过程的时间/内存消耗
indexed_at: 2026-02-27T11:44:46.806318+00:00
---

用户要求我分析这篇关于诊断 GRPO 训练过程时间/内存消耗的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 主要内容是关于使用 SwanLab 进行性能监控
2. 涉及 ms-swift 框架中的 GRPO 训练过程
3. 包括日志统一格式化的代码
4. 涉及 VLLM 中的时间记录分析

标签应该包括：GRPO 训练、性能监控、SwanLab、ms-swift、日志分析等

摘要需要概括核心内容

关键概念包括：SwanLab、GRPO、ms-swift、patch_profiling_context、VLLM 等

关联笔记：从提供的笔记列表来看，这些都是 2021 年的笔记，主要是关于 MySQL、Clickhouse、数据库相关的内容，与这篇 2025 年关于 AI 训练性能监控的笔记没有直接关联。所以关联笔记应该是「无」。

让我按照格式输出。
## 标签
GRPO 训练，性能监控，SwanLab，ms-swift，日志分析，内存诊断

## 摘要
记录使用 SwanLab 对 ms-swift 框架中 GRPO 训练过程进行性能 profiling 的配置方法。包括统一日志格式、标记关键代码块（generate、reward function、compute_loss 等）以分析时间/内存消耗。

## 关键概念
- SwanLab: 用于机器学习的实验跟踪和性能监控平台
- GRPO: 一种强化学习训练算法，用于大语言模型训练
- patch_profiling_context: ms-swift 中用于标记和监控代码块性能的工具
- ms-swift: 魔搭社区的大模型微调框架

## 关联笔记
无
