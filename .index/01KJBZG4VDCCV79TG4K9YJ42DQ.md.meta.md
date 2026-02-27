---
note: 01KJBZG4VDCCV79TG4K9YJ42DQ.md
title: 20241006 - 阅读论文*: Planning in the Dark: LLM-Symbolic Planning Pipeline without Experts
indexed_at: 2026-02-27T11:17:23.490119+00:00
---

用户要求我分析这篇关于论文阅读的笔记，生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：这是一篇关于论文《Planning in the Dark: LLM-Symbolic Planning Pipeline without Experts》的阅读笔记

2. **核心内容**：
   - 论文目的是解决自动规划任务中人为生成的行动模式过于单一且成本高的问题
   - 介绍了共形预测（Conformal Prediction）概念
   - 论文主要思路：通过 LLM 多次生成 + 过滤实现行动模式多样化，使用 PDDL 作为形式化
   - 介绍了 PDDL（规划领域定义语言）的概念

3. **关键概念**：
   - 行动模式 (Action Schema)
   - 规划 (Plan)
   - 符号规划器 (Symbolic Planner)
   - 共形预测 (Conformal Prediction)
   - PDDL（规划领域定义语言）
   - LLM（大语言模型）

4. **标签**：应该是关于 AI 规划、LLM、符号规划、PDDL、共形预测等

5. **关联笔记**：查看提供的其他笔记列表，都是关于数据库、MySQL、Clickhouse、系统调试等技术内容，与这篇 AI 规划主题的笔记没有明显关联

让我生成符合格式的输## 标签
LLM, 符号规划, PDDL, 共形预测, 行动模式, 自动规划

## 摘要
本文阅读笔记介绍了利用 LLM 生成多样化行动模式的论文，通过语义一致性过滤和共形预测筛选候选方案。行动模式以 PDDL 形式化，输入经典符号规划器生成规划方案并按语义相似度排序。

## 关键概念
- 行动模式 (Action Schema): 定义动作的抽象模板，规定参数、前提条件和效果
- 共形预测 (Conformal Prediction): 为预测结果提供置信度评估的方法，分布无关且有限样本有效
- PDDL: 规划领域定义语言，用于形式化描述规划问题的领域和问题实例
- 符号规划器 (Symbolic Planner): 接收行动模式和输入，搜索可行规划方案的算法

## 关联笔记
无
