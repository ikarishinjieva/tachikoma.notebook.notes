---
note: 01KJBZ9ZBKQ9FA70P9KBRTR59V.md
title: 20240613 - 阅读论文: How can we know When language models know? on the calibration of language models for question answering
indexed_at: 2026-03-05T10:08:39.660829+00:00
---

## 摘要
论文讨论 LLM 答案可靠性问题，提出多种校准方法（微调、后处理）。重点讲解基于特征的决策树校准法：通过提取模型预测概率、答案特征、问题特征等，训练决策树预测更准确的置信度分数，实现对 LLM 输出的可靠性评估。

## 关键概念
- 校准 (Calibration): 对 LLM 答案的正确性进行概率校准，使置信度与实际准确率一致
- 基于特征的决策树: 利用模型预测结果和相关特征训练独立校准模型，不修改原 LLM
- 特征提取: 从模型预测概率、答案长度、问题类型等维度提取反映可靠性的特征
- 温度缩放 (Temperature Scaling): 基于温度的后处理校准方法，调整输出概率分布
- 置信度标签: 通过比较模型预测与真实答案得到的二元标签（正确=1，错误=0）

## 关联笔记
- 01KJBZ9R4MN3T2EN6DV5JDPR7F.md: 讨论 LLM 不确定性度量方法，指出自置信度易过度自信的问题，与本校准主题呼应
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述笔记，在相关工作中引用了本论文
