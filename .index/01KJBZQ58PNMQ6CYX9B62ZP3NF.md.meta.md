---
note: 01KJBZQ58PNMQ6CYX9B62ZP3NF.md
title: 20250108 - ms-swift会缓存训练数据, 训练数据变更时可能无法响应
indexed_at: 2026-02-27T11:26:29.234687+00:00
---

用户希望我分析这篇关于 ms-swift 缓存训练数据的笔记，并生成索引元数据。让我分析笔记内容：

笔记标题：20250108 - ms-swift 会缓存训练数据，训练数据变更时可能无法响应

笔记内容：
- 存储目录在~/.cache/huggingface/datasets/json
- 通过环境变量 HF_DATASETS_CACHE="" 将其禁用

这是一个关于机器学习训练工具 ms-swift 的技术笔记，涉及 HuggingFace datasets 的缓存机制。

其他笔记列表主要是关于数据库（MySQL、Clickhouse）、系统性能、GDB 调试等主题，与这篇笔记的主题（机器学习训练数据缓存）没有明显关联。

让我生成索引：

## 标签
ms-swift, 训练数据，缓存，HuggingFace, 机器学习

## 摘要
记录 ms-swift 框架会缓存训练数据到本地目录，当训练数据变更时可能无法响应更新。可通过设置 HF_DATASETS_CACHE 环境变量禁用缓存机制。

## 关键概念
- HF_DATASETS_CACHE: HuggingFace datasets 缓存目录的环境变量
- 缓存目录: ~/.cache/huggingface/datasets/json，存储训练数据缓存
- 环境变量配置: 通过设置空字符串禁用缓存功能

## 关联笔记
无（其他笔记主要是数据库、系统性能相关，与机器学习训练缓存无直接关联）
## 标签
ms-swift, 训练数据，缓存，HuggingFace, 机器学习

## 摘要
记录 ms-swift 框架会缓存训练数据到本地目录，当训练数据变更时可能无法响应更新。可通过设置 HF_DATASETS_CACHE 环境变量禁用缓存机制。

## 关键概念
- HF_DATASETS_CACHE: HuggingFace datasets 缓存目录的环境变量
- 缓存目录: ~/.cache/huggingface/datasets/json，存储训练数据缓存
- 环境变量配置: 通过设置空字符串禁用缓存功能

## 关联笔记
无
