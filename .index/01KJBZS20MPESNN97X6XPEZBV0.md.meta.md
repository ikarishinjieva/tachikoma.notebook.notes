---
note: 01KJBZS20MPESNN97X6XPEZBV0.md
title: 20250318 - 阅读论文*: Chain of Draft: Thinking Faster by Writing Less
indexed_at: 2026-03-05T11:34:13.057288+00:00
---

## 摘要
Chain of Draft (CoD) 是一种优化推理效率的提示方法，要求模型每步推理仅输出最多 5 个关键词句而非完整句子。该方法在保持推理方向控制的同时减少 token 消耗和延迟，通过简洁的核心算式替代冗长解释。

## 关键概念
- Chain of Draft (CoD): 每步推理只保留 5 个单词以内的关键词句，减少冗余信息
- Chain of Thought (CoT): 传统的逐步推理方法，输出完整的思考过程和解释
- 最小化草稿: CoD 的核心理念，保留最关键信息的同时控制思考发散
- 分隔符答案提取: 使用 #### 等分隔符标记最终答案，便于程序化提取

## 关联笔记
- 01KJBZYC8BKTFV07JJNAQNS91T.md: 同为大模型推理机制研究论文，涉及 Chain-of-thought Faithfulness 分析
- 01KJBZP68GF2Z83W0EP4DT7V3H.md: CCOT 论文同样研究压缩思维链以提高推理效率
- 01KJC09VMCC3G9MT6AZPAWAB54.md: 实际项目中应用 CoT 流程提升判断准确率的工程实践
