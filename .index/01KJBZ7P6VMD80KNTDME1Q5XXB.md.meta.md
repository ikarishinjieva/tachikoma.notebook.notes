---
note: 01KJBZ7P6VMD80KNTDME1Q5XXB.md
title: 20240323 - 对embedding进行压缩
indexed_at: 2026-03-05T09:40:58.621449+00:00
---

## 摘要
探讨了 embedding 压缩的多种方法（PCA、自编码器、量化等），但在训练 BGE-M3 时发现应压缩文本而非向量。转向使用 LLM 重写文本来减小输入维度，提升微调效果。

## 关键概念
- embedding 压缩：通过降维方法减小向量维度，降低存储和计算开销
- BGE-M3: 将文本转换为向量的 embedding 模型
- 文本重写：使用 LLM 压缩文本内容，减小输入维度
- 主成分分析 (PCA): 线性降维方法，保留数据主要特征
- 自编码器：神经网络模型，学习数据的压缩表示

## 关联笔记
- 01KJBZ68S0R9NS2HK7MWN06G5A.md: 20240227 对 embedding 进行微调的初始实验记录
- 01KJBZ6V4HFTNCBMEZSA71YAKB.md: 20240228 增加训练数据的微调实验
- 01KJBZ72ZM9JSDEGSMRZ393PP1.md: 20240304 评估数据集的微调笔记
