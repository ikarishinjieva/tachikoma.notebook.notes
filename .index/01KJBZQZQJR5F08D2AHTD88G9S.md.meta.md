---
note: 01KJBZQZQJR5F08D2AHTD88G9S.md
title: 20250131 - 阅读论文*: META-CHUNKING: LEARNING EFFICIENT TEXT SEGMENTATION VIA LOGICAL PERCEPTION
indexed_at: 2026-03-05T11:17:42.059236+00:00
---

## 摘要
论文提出使用困惑度 (PPL) 拐点识别文本逻辑边界作为分块点，基于逻辑连贯文本内部困惑度低平稳、逻辑断点处困惑度上升的原理。提供两种策略：直接 PPL 分块按阈值切分，结合动态合并的 PPL 分块先产生元分块再按需合并以控制长度。

## 关键概念
- 困惑度 (PPL): 评估 GT 文本与 LLM 续写文本之间的概率差，反映模型对文本的困惑程度
- 困惑度拐点: 困惑度突然上升的点，标志逻辑或主题转换，可作为分块边界
- 直接 PPL 分块: 根据预设困惑度阈值直接分块，句子间差异超阈值则作为新块起点
- 动态合并分块: 先以低阈值生成元分块，再按长度需求合并相邻元分块
- 逻辑一致性: 连贯文本块内部句子间困惑度低且平稳的特性

## 关联笔记
- 01KJBZRFKPHCGB3ETQ4MS9CVH0.md: 同样使用困惑度 (PPL) 阈值评估数据质量，将 PPL 用于筛选推理路径
- 01KJBZFPM6BX07HW9FQM0QZ670.md: 分析 dsRAG 项目中的 chunking 创新，涉及语义切片和分块策略
