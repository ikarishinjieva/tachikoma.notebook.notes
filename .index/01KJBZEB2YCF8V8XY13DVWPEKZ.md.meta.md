---
note: 01KJBZEB2YCF8V8XY13DVWPEKZ.md
title: 20240815 - 阅读论文*: Learning to Generate Prompts for Dialogue Generation through Reinforcement Learning
indexed_at: 2026-03-05T10:32:30.932989+00:00
---

## 标签
强化学习，提示生成，对话生成，大模型优化，小模型训练，成本优化

## 摘要
提出通过训练一个小模型使用强化学习来生成提示词，再用提示词调用大模型生成输出的方法。该方法避免直接微调大模型的高成本，只需训练小模型即可间接改变大模型行为。

## 关键概念
- 强化学习: 用于训练小模型根据输入生成合适提示词的算法
- 提示生成: 小模型根据输入内容自动生成优化提示词的过程
- 大模型行为控制: 通过提示词间接改变大模型输出，避免直接微调
- 小模型训练: 训练成本低、迭代快，适合强化学习试错

## 关联笔记
- 01KJC06PG4RM9T7WFDEN5HPZPN.md: 讨论强化学习和模仿学习在不同任务中的应用策略对比
- 01KJBZYAS3THKM4CYDK96M7AQV.md: 讨论大模型后训练中提示选择和 RL 训练策略
- 01KJBZY39GJE38CB7BYSTRWMBX.md: 讨论 GRPO 训练中 prompt 长度对训练效率的影响
