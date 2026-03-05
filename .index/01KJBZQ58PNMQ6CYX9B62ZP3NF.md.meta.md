---
note: 01KJBZQ58PNMQ6CYX9B62ZP3NF.md
title: 20250108 - ms-swift会缓存训练数据, 训练数据变更时可能无法响应
indexed_at: 2026-03-05T11:11:27.789948+00:00
---

## 摘要
ms-swift 会将训练数据缓存在 `~/.cache/huggingface/datasets/json` 目录。当训练数据发生变更时，缓存可能导致无法响应最新数据。可通过设置环境变量 `HF_DATASETS_CACHE=""` 禁用缓存机制。

## 关键概念
- HF_DATASETS_CACHE: 控制 Huggingface 数据集缓存的环境变量
- huggingface datasets cache: Huggingface 数据集的本地缓存目录
- ms-swift: ModelScope 的 Swift 训练框架

## 关联笔记
- 01KJBZQ6DWHS6EB67JVEPB2H98.md: 同一天 (20250108) 的 ms-swift 微调训练笔记
- 01KJBZRGRZ1WMJ4QQSBPPYVYJN.md: 研究 ms-swift 训练数据收敛的笔记
- 01KJBZWQZ9BA47ET5VQBKGXEVE.md: 分析 ms-swift 的 GRPO compute_loss 函数
