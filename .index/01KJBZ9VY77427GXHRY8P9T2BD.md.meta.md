---
note: 01KJBZ9VY77427GXHRY8P9T2BD.md
title: 20240612 - 阅读论文: Ring: Repair Is Nearly Generation: Multilingual Program Repair with LLMs
indexed_at: 2026-03-05T10:05:40.685321+00:00
---

## 摘要
Ring 论文提出了一种基于 LLM 的多语言程序修复方法，采用类似 RAG 的检索 - 生成 - 排序流程。核心创新在于使用错误代码进行检索（更专注问题定位）和基于 token 概率对数和的排序选择（衡量 LLM 对修复方案的信心）。

## 关键概念
- 错误代码检索: 使用报错代码而非自然语言进行检索，使检索更聚焦于问题本身
- Token 概率对数和: 用 LLM 生成修复代码时每个 token 概率的对数和作为排序依据，反映模型信心
- 修复代码排序: 从多个候选修复方案中选择对数和最高的片段作为最终输出

## 关联笔记
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 论文笔记，同样结合检索增强与推理，使用反射标记触发检索
- 01KJBZG9HE3MZ0312PBHKXC0M7.md: Speculative RAG 论文笔记，涉及多草稿生成与择优选择的排序思路
- 01KJBYXQ5GXWFSE6D0CVP2MDXM.md: RAG 相关笔记，可对比检索策略的异同
