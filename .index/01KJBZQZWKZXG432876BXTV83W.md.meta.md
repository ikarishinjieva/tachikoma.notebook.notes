---
note: 01KJBZQZWKZXG432876BXTV83W.md
title: 20250131 - 阅读论文*: ZEUS: Enhancing Zero-shot Chain of Thought Prompting via Uncertainty-Guided Strategy Selection
indexed_at: 2026-03-05T11:17:58.011364+00:00
---

## 标签
零样本思维链，不确定性估计，示例选择，提示工程，大语言模型，推理增强

## 摘要
ZEUS 通过多重扰动（温度、提示词、问题改写）估计问题不确定性，基于不确定性筛选高质量样本构建 few-shot demonstrations。该方法无需人工标注，从无标签数据集中自动选择中等不确定性样本，显著提升零样本 CoT 推理性能。

## 关键概念
- 不确定性估计：使用预测熵衡量模型对问题的不确定程度，越高表示越不确定
- 多重扰动：通过温度扰动、提示词扰动、问题改写三种方式对同一问题生成多个答案
- 样本选择策略：基于不确定性均值 (µ) 和标准差 (σ) 选择特定范围 (umin, umax) 的样本
- Few-shot demonstrations：用筛选后的问题 - 推理步骤 - 答案三元组构建推理示例
- 聚类代表选择：使用 k-Means++ 聚类后从每类选择最接近中心的问题作为代表

## 关联笔记
- 01KJBZWP1711J31GDNMNTAQW01.md: 讨论 CoT 推理中的熵模式和高熵词元对推理性能的影响，与不确定性估计原理相通
- 01KJBZRE950XFNR96B42N1J08E.md: 涉及使用置信度评估模型不确定性的方法，与 ZEUS 的不确定性度量目标一致
- 01KJBZPHPP830PMYWNX64W2A4H.md: 探讨 SoT 作为 CoT 的替代推理方法，同属提升 LLM 推理能力的 prompting 策略
