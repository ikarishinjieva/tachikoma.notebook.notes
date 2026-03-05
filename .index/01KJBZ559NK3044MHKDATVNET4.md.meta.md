---
note: 01KJBZ559NK3044MHKDATVNET4.md
title: 20240131 - 阅读论文: Reinforcement Learning for Optimizing RAG for Domain Chatbots
indexed_at: 2026-03-05T09:28:23.965467+00:00
---

## 标签
RAG, 强化学习, 嵌入模型, infoNCE, 聊天机器人, 成本优化

## 摘要
论文提出用强化学习优化 RAG 聊天机器人的 LLM 调用成本，通过策略模型选择是否检索 FAQ 上下文。结合内部训练的嵌入模型（infoNCE 损失微调）和相似性阈值，实现约 31% 令牌节省并略微提升准确性。

## 关键概念
- infoNCE 损失函数: 对比学习方法，通过最大化正样本对相似性并归一化负样本对来训练嵌入模型
- RL 优化: 使用强化学习策略模型与 RAG 流水线交互，选择获取上下文或跳过检索以优化成本
- 相似性阈值: 通过设定相似度阈值减少不必要的 LLM 调用，降低令牌消耗
- 嵌入模型微调: 使用 e5-base-v2 初始化，基于 ChatGPT 和手动标注数据用 infoNCE 损失训练

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: RAG 综述论文，同样提到 infoNCE 损失函数用于 retriever 微调
- 01KJBZ5MJTQ4XWXMBSZEB3QWH0.md: SELF-RAG 论文，同样使用强化学习/反思机制优化 RAG 检索策略
