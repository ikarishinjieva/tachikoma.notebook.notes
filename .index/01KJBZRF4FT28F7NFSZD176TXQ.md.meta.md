---
note: 01KJBZRF4FT28F7NFSZD176TXQ.md
title: 20250211 - 阅读论文*: V-STaR: Training Verifiers for Self-Taught Reasoners
indexed_at: 2026-02-27T11:33:25.728569+00:00
---

用户要求我分析一篇关于 V-STaR 论文的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：V-STaR: Training Verifiers for Self-Taught Reasoners 论文阅读
2. 核心内容：
   - STaR 方法：用正确用例训练模型
   - V-STaR 方法：增加验证器，用 DPO 训练验证器
   - 两者的区别：验证器、错误答案利用、训练数据更新方式、迭代次数、生成答案数量、参数更新方式

3. 标签应该包括：V-STaR、STaR、验证器、DPO、大模型训练等

4. 摘要：概括 V-STaR 与 STaR 的核心区别和改进

5. 关键概念：STaR、V-STaR、DPO、验证器等

6. 关联笔记：从提供的笔记列表中看，这些都是关于数据库、系统性能、Clickhouse 等主题的笔记，与这篇 AI/大模型论文笔记没有明显关联，所以应该写「无」

让我按照要求的格式输出。## 标签
V-STaR, STaR, 验证器训练, DPO, 大模型推理, 论文阅读

## 摘要
V-STaR 在 STaR 方法基础上引入验证器，通过 DPO 利用错误答案训练裁判模型。相比 STaR，V-STaR 在数据利用效率、迭代次数和答案选择策略上都有改进，实现性能提升。

## 关键概念
- STaR: 利用模型自身生成的正确答案迭代改进模型，无独立验证器
- V-STaR: 引入验证器判断答案正误，用 DPO 训练验证器区分对错答案
- DPO: 直接偏好优化，用于训练验证器区分正确答案和错误答案
- LoRA: 参数高效微调方法，V-STaR 论文中采用的参数更新方式

## 关联笔记
无
