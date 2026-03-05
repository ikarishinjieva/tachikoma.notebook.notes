---
note: 01KJBYH6Z36KSJE0C1YDGDDQKD.md
title: 20220213 - 向量数据库性能测试用例集
indexed_at: 2026-03-05T07:55:42.187286+00:00
---

## 摘要
记录使用 Milvus 的 SIFT 数据集进行向量数据库性能测试的准备工作。重点研究特征二值化方法，包括 LSHBOX 工具编译和 ITQ 算法原理分析。

## 关键概念
- SIFT 数据集: Milvus 提供的标准基准测试数据集，用于向量检索性能评估
- ITQ (Iterative Quantization): 迭代量化算法，通过旋转矩阵将数据投影后二值化为 -1/+1
- LSH (Locality-Sensitive Hashing): 局部敏感哈希，将相似向量映射到同一哈希桶的概率方法
- 二值化: 将实值向量转换为二值向量的技术，用于减少存储空间和加速检索

## 关联笔记
- 01KJBYGX27WHQYN736YE9ENREM.md: 同主题论文学习笔记，详细讲解 SIFT、LSH、ITQ 等向量二值化算法
- 01KJBZ7P6VMD80KNTDME1Q5XXB.md: embedding 压缩方法总结，包含 LSH、PQ 等向量降维技术
