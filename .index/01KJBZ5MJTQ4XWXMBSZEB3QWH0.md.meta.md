---
note: 01KJBZ5MJTQ4XWXMBSZEB3QWH0.md
title: 20240205 - 阅读论文: SELF-RAG: LEARNING TO RETRIEVE, GENERATE, AND CRITIQUE THROUGH SELF-REFLECTION
indexed_at: 2026-03-05T09:31:34.607810+00:00
---

## 标签
SELF-RAG, 检索增强生成, 自我反思, 反思标记, 语言模型, 属性问题回答

## 摘要
SELF-RAG 是一种通过检索和自我反思来提高语言模型生成质量的框架，可自适应决定何时检索并使用反思标记评估内容。该框架训练 Critic 和 Generator 两个模型，通过 ISREL、ISSUP、ISUSE 等标记对检索段落和生成输出进行细粒度评估与排序。

## 关键概念
- SELF-RAG: 自反思检索增强生成框架，通过检索和自我反思提高生成准确性和相关性
- 反思标记 (Reflection Tokens): 特殊标记用于评估检索段落和生成输出的质量
- ISREL: 判断检索到的文本段落是否为输入提供了有效信息
- ISSUP: 判断生成的输出每一部分是否都有检索证据支撑
- ISUSE: 评估生成的输出对输入问题的有用程度/匹配程度

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文，在技术分类部分引用并讨论了 SELF-RAG 方法
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 同样使用反射标记控制检索和评估，与 SELF-RAG 设计理念相似
- 01KJBZG3NMEV9RPF9NN5VBTQ9Z.md: RAGCHECKER 是评估 RAG 系统生成质量的框架，可用于评估 SELF-RAG 效果
