---
note: 01KJBZWJBAGGD1TA8SGM6YZXR4.md
title: 20250804 - 阅读论文*: KAG-Thinker
indexed_at: 2026-03-05T11:49:57.568200+00:00
---

## 摘要
KAG-Thinker 是一种结合知识检索与逻辑推理的框架，核心创新包括知识边界判定（先生成后评估策略）、广度分解（一次性规划子问题）和深度求解（迭代检索过滤）。通过医疗诊断案例展示其如何用逻辑形式组织思考步骤，实现可追溯的个性化决策。

## 关键概念
- 知识边界判定 (Knowledge Boundary Determination): 采用"先生成，后评估"策略，通过双重验证决定是否启动外部检索
- 广度分解 (Breadth Decomposition): 一次性列出所有子问题并创建逻辑形式的情报卡片，制定完整作战总纲
- 深度求解 (Depth Solving): 针对单个子问题通过"自省→决策→检索→过滤→迭代"循环获取精准答案
- 聚焦与推理 (Focusing and Reasoning): 过滤检索噪声，提炼核心知识并与内部知识结合
- 逻辑形式 (Logical Form): 用自然语言 (Step) 和机器指令 (Action) 双语表达情报卡片

## 关联笔记
- 01KJBZWKEDHZEEK4TMX2V4XZ12.md: ChatDoctor 同为医疗领域检索增强方案，采用微调+RAG 架构解决医疗问答
- 01KJBZRE950XFNR96B42N1J08E.md: OPEN-RAG 同样结合检索与推理，使用反射标记控制检索时机和评估检索质量
