---
note: 01KJBZQT2DD1YNRJ2J49PP9MN0.md
title: 20250122 - 使用ms-swift对Qwen2-VL进行微调 [19] - 调优长度7的效果[3]
indexed_at: 2026-03-05T11:14:50.193515+00:00
---

## 摘要
记录使用 ms-swift 对 Qwen2-VL 进行微调时增加 ring 结构规范数据的实验，包含 chain/branch/ring 等多种分子结构的数据集配置。数据按长度 2-6 分类，涵盖 raw 和 enhanced_by_image 两个版本，用于提升模型对环状分子结构的识别能力。

## 关键概念
- ring_selfies: 环状分子的 SELFIES 表示法数据集，用于训练模型识别环状结构
- enhanced_by_image: 通过图像增强处理的数据版本，提升视觉 - 化学表达关联
- 规范数据: 满足 selfies+smiles 幂等性的标准化训练数据，确保分子表达唯一性

## 关联笔记
- 01KJBZRRGYWH5WHAZ2TZKMP91F.md: 同一微调系列的后续笔记，记录工作移交和实验路径梳理
- 01KJBZN4KQV7NK5GRT46165QTR.md: 微调知识整理，包含 ms-swift 微调 Qwen2-VL 的基础知识和重要概念
- 01KJBZJSM5YEKX18QPBPFMAYNX.md: 早期 Qwen2-VL 微调过拟合分析，涉及模型评估和训练曲线
