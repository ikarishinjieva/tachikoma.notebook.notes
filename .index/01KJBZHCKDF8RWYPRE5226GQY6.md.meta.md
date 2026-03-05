---
note: 01KJBZHCKDF8RWYPRE5226GQY6.md
title: 20241021 - 阅读论文*: cddd: Learning continuous and data-driven molecular descriptors by translating equivalent chemical rep
indexed_at: 2026-03-05T10:49:49.186909+00:00
---

## 摘要
论文提出 cddd 分子描述符生成方法，使用类似翻译的自编码器架构（分子表示 1→中介向量→分子表示 2）训练连续向量表示。训练数据来自 ZINC15 和 PubChem 合并去重后的约 7200 万化合物，经 RDKit 过滤确保质量。

## 关键概念
- cddd: 连续数据驱动的分子描述符，用作不同分子表示之间的中介向量
- 自编码器架构: 编码器将分子映射到中介表示，解码器从中介表示还原分子
- 分子表示转换: 类似语言翻译，通过不同分子表示间的转换训练中介向量
- RDKit 过滤: 用于数据质量控制，包括分子量、重原子数、logP 等条件

## 关联笔记
- 01KJBZGYAPQFV3CQ1A8H479FZ7.md: Img2Mol 论文笔记，将图片到 SMILES 任务拆解为图片到 CDDD 向量+CDDD 向量到 SMILES
- 01KJBZH3679KRGP1N7E91WZR91.md: Img2Mol 训练笔记，涉及 cddd 训练数据使用和模型验证
- 01KJBZGZS514G9FB33XS2BKZ97.md: Img2Mol 测试笔记，记录 cddd_server 使用和本地模型部署
