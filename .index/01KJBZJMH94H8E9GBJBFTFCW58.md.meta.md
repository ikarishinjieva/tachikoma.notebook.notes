---
note: 01KJBZJMH94H8E9GBJBFTFCW58.md
title: 20241108 - 尝试输出transformers的evaluate结果
indexed_at: 2026-02-27T11:21:17.679227+00:00
---

用户需要我分析这篇笔记并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20241108 - 尝试输出 transformers 的 evaluate 结果
2. 主要内容：关于在训练化学分子式识别模型时，尝试使用 transformers 的 evaluate 功能，特别是 predict_with_generate 参数的配置和调试过程

核心内容：
- 背景：训练模型时发现过早过拟合，想查看训练过程中的 evaluate 输出
- 调查了 transformers trainer.py 中的 evaluate 调用链
- 详细解释了 predict_with_generate 参数的两种评估方式
- 尝试开启 predict_with_generate 时遇到的问题和解决方案
- 最终的配置文件和训练效果

标签应该包括：transformers, evaluate, predict_with_generate, 模型训练，Qwen2-VL, Llama-factory 等

摘要：记录在训练化学分子式识别模型时，配置 transformers 的 evaluate 功能和 predict_with_generate 参数的过程。解决了 Llama-factory 参数检查和 Qwen2-VL 模型兼容性等问题。

关键概念：
- predict_with_generate: 决定使用基于生成序列的评估指标还是基于 token 预测概率的评估
- evaluate: transformers 训练过程中的评估功能
- Llama-factory: 模型微调框架


- Qwen2-VL: 视觉语言模型

关联笔记：从其他笔记列表来看，没有直接相关的笔记（其他笔记都是 2021 年的 MySQL、Clickhouse 等内容，与这篇 2024 年的 ML 训练笔记无关）## 标签
transformers, evaluate, predict_with_generate, 模型训练, Qwen2-VL, Llama-factory

## 摘要
记录在训练化学分子式识别模型时，配置 transformers 的 evaluate 功能和 predict_with_generate 参数的调试过程。解决了 Llama-factory 参数检查逻辑错误和 Qwen2-VL 模型 tokenizer 兼容性等问题，最终成功开启序列生成评估。

## 关键概念
- predict_with_generate: 决定使用基于生成序列的整体质量评估还是基于 token 预测概率的逐位评估
- evaluate: transformers 训练过程中用于评估模型性能的机制
- Llama-factory: 大模型微调框架，本笔记中对其代码进行了修改以支持特定配置

## 关联笔记
无
