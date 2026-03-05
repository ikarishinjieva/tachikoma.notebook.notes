---
note: 01KJBZ72ZM9JSDEGSMRZ393PP1.md
title: 20240304 - 对embedding进行微调 - eval dataset
indexed_at: 2026-03-05T09:38:45.980341+00:00
---

## 标签
需要 3-6 个关键词：Transformer, Trainer, eval_dataset, 模型微调，验证集，学习率调度器

## 摘要
2-3 句话概括核心内容

## 关键概念
3-5 个核心概念

## 关联笔记
列出 0-3 篇关联笔记## 标签
Transformer, Trainer, eval_dataset, 模型微调，验证集，学习率调度器

## 摘要
解析 Hugging Face Trainer 中 eval_dataset 的作用及其在训练循环中的影响机制。通过源码分析确认 eval_dataset 用于评估模型性能，并可联动 ReduceLROnPlateau 调度器动态调整学习率。

## 关键概念
- eval_dataset: Trainer 中用于训练过程中监控模型泛化能力的验证数据集
- _maybe_log_save_evaluate: Trainer 训练循环中负责日志记录、评估和保存检查点的核心方法
- ReduceLROnPlateau: 根据验证集性能指标动态调整学习率的调度器
- 早停机制: 基于验证集性能不再提升时终止训练的策略

## 关联笔记
- 01KJBZ68S0R9NS2HK7MWN06G5A.md: 同一 embedding 微调系列的前一篇，记录微调任务启动
- 01KJBZ6V4HFTNCBMEZSA71YAKB.md: 同一 embedding 微调系列，讨论训练数据扩充与校验集构造
- 01KJBZ7P6VMD80KNTDME1Q5XXB.md: 同一 embedding 微调系列，探讨微调数据的维度压缩问题
