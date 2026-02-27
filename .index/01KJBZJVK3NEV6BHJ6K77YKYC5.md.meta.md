---
note: 01KJBZJVK3NEV6BHJ6K77YKYC5.md
title: 20241114 - 使用ms-swift对Qwen2-VL进行微调 [2] - 使用SELFIES替代SMILES
indexed_at: 2026-02-27T11:21:42.370672+00:00
---

用户要求我分析一篇笔记并生成索引元数据。让我仔细分析笔记内容：

笔记标题：20241114 - 使用 ms-swift 对 Qwen2-VL 进行微调 [2] - 使用 SELFIES 替代 SMILES

笔记内容：
- 使用脚本生成基于 SMILES 的 COT
- 有一个附件：add_decimer_image_cot_SELFIES.py
- 发现原始数据中有一些 SMILES 标记是错误的

核心主题：
1. Qwen2-VL 模型微调
2. SMILES 和 SELFIES（化学分子表示方法）
3. COT（Chain of Thought，思维链）
4. 数据质量问题（SMILES 标记错误）

标签应该是：Qwen2-VL, 微调，SMILES, SELFIES, 化学分子，COT

摘要：记录使用 ms-swift 框架对 Qwen2-VL 视觉语言模型进行微调的过程。主要工作是用脚本生成基于 SMILES 的化学分子 COT 数据，并发现原始数据存在 SMILES 标记错误的问题。

关键概念：
- SMILES: 化学分子的一种字符串表示方法
- SELFIES: 一种替代 SMILES 的化学分子表示方法，更健壮
- Qwen2-VL: 阿里云的视觉语言模型
- COT (Chain of Thought): 思维链，一种让模型展示推理过程的方法


- ms-swift: 模型微调框架

关联笔记：检查提供的笔记列表，这些都是 2021 年的笔记，主要涉及 MySQL、Clickhouse、系统性能等主题，与这篇关于 Qwen2-VL 微调和化学分子表示的笔记没有明显关联。

输出结果。## 标签
Qwen2-VL, 模型微调, SMILES, SELFIES, 化学分子, COT

## 摘要
记录使用 ms-swift 框架对 Qwen2-VL 视觉语言模型进行微调的进程。主要工作是通过脚本生成基于 SMILES 的化学分子思维链（COT）数据，并发现原始数据中存在 SMILES 标记错误的问题。

## 关键概念
- SMILES: 化学分子的一种线性字符串表示方法
- SELFIES: 一种替代 SMILES 的化学分子表示方法，具有更好的鲁棒性
- Qwen2-VL: 阿里云开源的视觉语言大模型
- COT (Chain of Thought): 思维链，让模型展示逐步推理过程的方法

## 关联笔记
无
