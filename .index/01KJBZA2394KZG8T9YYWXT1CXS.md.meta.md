---
note: 01KJBZA2394KZG8T9YYWXT1CXS.md
title: 20240614 - 阅读论文*: Retrieve only when it needs: Adaptive retrieval augmentation for hallucination mitigation in large lan
indexed_at: 2026-03-05T10:09:17.088367+00:00
---

## 摘要
提出 Rowen 方法，通过检测相同查询在不同语言间回答的不一致性来识别 LLM 幻觉。分析了跨语言幻觉差异的三大成因：训练数据偏差、模型结构差异、翻译信息损失。

## 关键概念
- Rowen: 利用跨语言答案不一致性来识别 LLM 幻觉的方法
- 跨语言不一致性: 相同语义问题在不同语言下产生不同回答的现象
- 训练数据偏差: 不同语言训练数据量和文化背景差异导致模型知识覆盖面不均
- 自适应检索增强: 仅在需要时进行检索以减少幻觉的 RAG 策略

## 关联笔记
- 01KJBZ88V83251ZJ7E8G9QBSV2.md: 检索增强相关论文索引，直接引用了 Rowen 论文
- 01KJBZA3WQ1ZCDF97VS6A0AVQ2.md: RAT 方法同样通过检索增强解决 LLM 幻觉问题
- 01KJBZ5SXG26R34S13HPP05SYY.md: ITER-RETGEN 研究检索 - 生成协同以增强 LLM 能力
