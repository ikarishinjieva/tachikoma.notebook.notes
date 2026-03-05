---
note: 01KJBZFXF5Q5Q83558J8SNY49X.md
title: 20241005 - 阅读论文*: LoT: Logic-of-Thought: Injecting Logic into Contexts for Full Reasoning in Large Language Models
indexed_at: 2026-03-05T10:44:50.971620+00:00
---

## 摘要
LoT 方法针对 CoT 推理中结论与过程冲突的"不忠实"问题，通过将文本转化为逻辑表达式进行演算，再将结果转回自然语言补充上下文。核心创新是用 Python 程序扩展逻辑关系以恢复隐含事实（如"哈利是人"、"书是知识载体"），解决纯形式化导致的信息丢失问题。

## 关键概念
- CoT 不忠实性: 思维链生成的推理过程与结论可能存在冲突或矛盾
- 逻辑形式化: 用 LLM 将自然语言文本转化为可演算的逻辑表达式
- 隐含事实丢失: 逻辑提取过程中忽略上下文默认但推理必需的背景知识
- 逻辑扩展: 使用 Python 程序对提取的逻辑关系进行推导和补充
- 自然语言回译: 将扩展后的逻辑关系翻译回自然语言作为上下文补充

## 关联笔记
- 01KJBZYC8BKTFV07JJNAQNS91T.md: 讨论 CoT 忠实性问题，区分模型书面推理与内部机制是否一致
- 01KJBZWP1711J31GDNMNTAQW01.md: 分析 CoT 推理中的熵模式，高熵词元作为推理分叉路口
- 01KJBZPHPP830PMYWNX64W2A4H.md: SoT 方法通过故事组织知识点激发推理，与 LoT 的逻辑组织形成对比
