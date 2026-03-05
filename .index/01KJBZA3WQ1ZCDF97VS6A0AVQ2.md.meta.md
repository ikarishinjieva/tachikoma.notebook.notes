---
note: 01KJBZA3WQ1ZCDF97VS6A0AVQ2.md
title: 20240614 - 阅读论文*: RAT: Retrieval Augmented Thoughts Elicit Context-Aware Reasoning in Long-Horizon Generation
indexed_at: 2026-03-05T10:10:56.147496+00:00
---

## 摘要
RAT（检索增强思维）通过迭代修正 CoT 生成的思维步骤，每步基于外部检索信息来确保推理准确性。该方法有效缓解长程生成任务中的幻觉问题，但无法修正初始 CoT 框架的结构性错误。

## 关键概念
- RAT: 检索增强思维，通过外部知识检索迭代修正思维链步骤
- 零样本 CoT: 不使用示例让 LLM 直接生成初始思维链
- 迭代修正: 逐一修正每个思维步骤，确保每步建立在可靠信息基础上
- 检索查询生成: 将任务指令和已修正步骤转换为检索查询
- 长程生成: 需要多步推理的复杂生成任务，易出现幻觉和推理错误

## 关联笔记
- 01KJBZYC8BKTFV07JJNAQNS91T.md: 同为大模型推理机制研究，涉及 CoT 忠实性分析和内部推理链追踪
- 01KJBZWP1711J31GDNMNTAQW01.md: 研究 CoT 推理中的熵模式，与 RAT 同属改进 LLM 推理质量的方法
- 01KJBZRPASEJVBQPE28AAP50YZ.md: 探讨 o1 模型的 reasoning path 构建，与 RAT 的迭代修正思路形成对比
