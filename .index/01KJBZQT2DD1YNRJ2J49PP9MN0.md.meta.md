---
note: 01KJBZQT2DD1YNRJ2J49PP9MN0.md
title: 20250122 - 使用ms-swift对Qwen2-VL进行微调 [19] - 调优长度7的效果[3]
indexed_at: 2026-02-27T11:27:29.793093+00:00
---

用户要求我分析一篇笔记并生成索引元数据。让我仔细阅读笔记内容：

笔记标题：20250122 - 使用 ms-swift 对 Qwen2-VL 进行微调 [19] - 调优长度 7 的效果 [3]

笔记内容主要是关于：
1. 增加 ring 的规范数据
2. 一个 commit: 66107a8679fe50aaa3604ae6794ccfbd1aeea43c
3. 大量的数据集文件路径，涉及：
   - chain_selfies
   - swap_atom_selfies_from_chain_selfies
   - branch_selfies
   - ring_selfies
   - 不同长度（length_2 到 length_6）
   - raw 和 enhanced_by_image 版本
   - cot_with_CNO_count 和 cot_with_CNO_count.enhance_by_mirror_selfies

这看起来是关于分子结构（SMILES/SELFIES 表示）的数据集整理，用于微调 Qwen2-VL 视觉语言模型。

标签应该包括：Qwen2-VL, 模型微调，分子数据集，SELFIES, 数据增强

摘要：记录使用 ms-swift 框架对 Qwen2-VL 模型进行微调的工作，主要增加了 ring 结构的规范训练数据。数据集包含多种分子表示形式（chain/branch/ring/swap_atom）和不同长度规格。

关键概念：
1. ms-swift: 模型微调框架
2. Qwen2-VL: 阿里通义千问视觉语言模型


3. SELFIES: 分子结构的字符串表示法
4. COT: Chain of Thought，思维链
5. 数据增强：通过镜像等方式增强训练数据

关联笔记：查看提供的其他笔记列表，这些都是关于 MySQL、Clickhouse、系统性能等的内容，与这篇分子 AI 微调的笔记没有直接关联。所以应该写「无」。## 标签
Qwen2-VL, 模型微调, 分子数据集, SELFIES, 数据增强, ms-swift

## 摘要
记录使用 ms-swift 框架对 Qwen2-VL 视觉语言模型进行微调的工作，主要增加了 ring 结构的规范训练数据。数据集涵盖多种分子表示形式（chain/branch/ring/swap_atom），按长度 2-6 分类，包含原始数据和镜像增强数据。

## 关键概念
- ms-swift: 大模型微调框架，支持多模态模型训练
- Qwen2-VL: 阿里通义千问视觉语言模型
- SELFIES: 分子结构的字符串表示法，比 SMILES 更规范
- COT: Chain of Thought，思维链训练数据
- 数据增强: 通过镜像等方式扩充训练数据集

## 关联笔记
无
