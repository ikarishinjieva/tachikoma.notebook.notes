---
note: 01KJBZEWQ67QAVXCD1J91HN23M.md
title: 20240826 - 阅读论文: Rethinking the Role of Demonstrations: What Makes In-Context Learning Work?
indexed_at: 2026-03-05T10:38:45.758313+00:00
---

## 摘要
论文研究发现 in-context learning 中模型性能对 demonstrations 的具体输入 - 标签映射不敏感，随机标签仍能取得优于 zero-shot 的性能。关键影响因素是输入文本分布、标签空间和 demonstrations 格式，而非标签正确性。

## 关键概念
- in-context learning: 通过少量输入 - 标签对 (demonstrations) 让模型完成新任务的学习方式
- demonstrations: 语境学习中提供给模型的输入 - 标签配对示例
- 随机标签方法: 将未标记输入与随机标签配对作为 demonstrations 的低成本方案
- 标签空间: 任务中可用标签的集合，对语境学习性能有重要影响
- 预训练先验知识: 模型在预训练阶段学到的语言统计规律和任务相关知识

## 关联笔记
- 01KJBZEXH23XZ79KS3W992P7ZA.md: 另一篇语境学习论文笔记，结论相反（大型模型能学习输入 - 标签映射）
- 01KJBZQZWKZXG432876BXTV83W.md: ZEUS 方法通过不确定性筛选构建高质量 few-shot demonstrations 提升 zero-shot CoT 性能
