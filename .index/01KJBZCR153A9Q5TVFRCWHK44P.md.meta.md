---
note: 01KJBZCR153A9Q5TVFRCWHK44P.md
title: 20240803 - 阅读论文: THINK-ON-GRAPH: DEEP AND RESPONSIBLE REASONING OF LARGE LANGUAGE MODEL ON KNOWLEDGE GRAPH
indexed_at: 2026-03-05T10:28:40.098856+00:00
---

## 摘要
论文提出 Think-on-Graph(ToG) 框架，实现 LLM 与 KG 的紧密耦合协同推理。ToG 通过 LLM 引导 KG 上的 beam search 迭代构建推理路径，ToG-R 则通过 LLM 预测关系并用 KG 验证，实现更高效的推理。

## 关键概念
- LLM+KG 范式: 从 KG 检索信息增强 prompt，LLM 仅作为翻译器，松耦合
- LLM*KG 范式: KG 与 LLM 在推理每步紧密协同，相互补充
- ToG 算法: LLM 结合问题文本评估选择 KG 中的边和点，迭代探索推理路径
- ToG-R 算法: LLM 预测下一步关系，用 KG 验证预测，更目标导向但精度略低

## 关联笔记
- 01KJBZJCR5P60FXY3BH47TAFH4.md: HippoRAG 同样使用知识图谱进行多跳推理检索
- 01KJBZBDWCWQHY5AD2FMAZEBFQ.md: CoE 算法也采用规划 - 查找循环在知识图谱中探索
- 01KJBZANGZVY92HGNK5652A6G8.md: PCST 算法用于知识图谱补全和关系路径预测
