---
note: 01KJBZA6J2T4E8E1FFNPE40RBY.md
title: 20240618 - ChatDBA: 如何判断参考材料和问题的相关性 [2]
indexed_at: 2026-03-05T10:12:52.056955+00:00
---

## 标签
ChatDBA, 相关性判断, MySQL 崩溃排查, 检索增强生成, 思维链, 诊断计划生成

## 摘要
研究 ChatDBA 系统中参考文档与用户问题的相关性判断问题，以"MySQL 执行 select 语句 crash"为例分析 AI 回答中 optimizer_switch 等内容与问题的间接推导关系。包含生成排查评估计划的完整 prompt 模板，要求使用 Python 伪代码形式输出诊断流程。

## 关键概念
- 相关性判断: 评估检索到的参考材料与用户问题的直接/间接关联程度
- 排查评估计划: 用结构化伪代码表示的数据库问题诊断流程
- 思维链: 通过 THINKING/FIND/IF 等函数构建逐步推理的诊断逻辑
- optimizer_switch: MySQL 优化器参数，与 crash 问题仅有间接关联
- 伪代码规范: SWITCH/Case/THINKING/FIND/IF/REASON_CONFIRMED/RETURN 函数体系

## 关联笔记
- 01KJBZA51RVTP0C0VXYHQ96C2D.md: 同一系列的相关性判断实验 [1]
- 01KJBZA6T5H1DQDV560RRNDGK7.md: 同一系列的相关性判断实验 [3]
- 01KJBZA7W62X0HJ2HGDXSYNQ7Z.md: 同一系列的相关性判断实验 [4]
