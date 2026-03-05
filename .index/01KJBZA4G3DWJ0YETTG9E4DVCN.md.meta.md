---
note: 01KJBZA4G3DWJ0YETTG9E4DVCN.md
title: 20240616 - ChatDBA: 诊断流程执行时产生模型幻觉
indexed_at: 2026-03-05T10:11:27.278572+00:00
---

## 摘要
记录 ChatDBA 在执行诊断流程代码时产生模型幻觉的案例。通过 MySQL 临时表 full 错误的诊断示例，展示诊断代码执行逻辑与人类输入信息的交互过程。

## 关键概念
- 模型幻觉: AI 在诊断流程中对未提供信息进行推测假设的错误行为
- 诊断流程代码: 包含 THINKING/FIND/IF/REASON_CONFIRMED/RETURN 函数的结构化排查逻辑
- 临时表空间: MySQL 中 innodb_temp_data_file_path 参数控制的临时数据存储区域

## 关联笔记
- 01KJBZAQ5MZVR57XF1YJQSGN57.md: 同系列诊断流程思维链方案探索笔记
- 01KJBZAGWAND5W9R05TTT38871.md: 包含相同的诊断流程结构说明（REASON_CONFIRMED 前后阶段划分）
- 01KJBZADENS4HHC18MT0EAZ182.md: ChatDBA 第二版发版前的结构梳理，涉及诊断流程设计
