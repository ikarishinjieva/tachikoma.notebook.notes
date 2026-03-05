---
note: 01KJBZG071EGFZEW1XATH1JV1T.md
title: 20241005 - 阅读论文*: REAP: Enhancing LLM Problem Solving with REAP: Reflection, Explicit Problem Deconstruction, and Advanc
indexed_at: 2026-03-05T10:45:09.905371+00:00
---

## 摘要
REAP 方法通过整合反思、显式问题解构和高级提示三个核心组件，解决现有 LLM 提示策略在复杂推理任务中的局限性。该方法包含三阶段流程（反思→问题解构→高级提示），引导模型进行系统性分析并通过迭代改进生成更准确、连贯、可解释的答案。

## 关键概念
- REAP 框架: 统一的动态上下文生成框架，整合反思、问题解构和高级提示
- 反思机制: 通过字面解释、严格解释、关键洞察检查等步骤防止模型偏离正确方向
- 显式问题解构: 全面提取问题特征、分解子问题、进行空间和对象分析
- 高级提示: 构建思维图、生成多种解决方案、识别最优解
- 思维图 (Graph of Thought): 用图结构表示问题，节点代表概念/状态，边代表关系

## 关联笔记
- 01KJBZ7SDESPWSX3VZV2NMJD8G.md: 提示词技巧笔记，引用了 REAP 论文作为相关方法
- 01KJBZD1P74N9Z643CH6RWT24X.md: REAPER 论文，同样使用基于推理的规划器增强 LLM 能力
- 01KJBZWP1711J31GDNMNTAQW01.md: 研究思维链推理中的熵模式，与推理增强主题相关
