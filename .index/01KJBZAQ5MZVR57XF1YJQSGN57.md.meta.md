---
note: 01KJBZAQ5MZVR57XF1YJQSGN57.md
title: 20240716 - ChatDBA: 用思维链的方案, 尝试生成排查图
indexed_at: 2026-03-05T10:21:51.330460+00:00
---

## 摘要
记录 ChatDBA 项目中使用思维链 (CoT) 方案生成 MySQL 故障排查图的实践。通过 MySQL crash 案例展示如何从思考过程自动生成结构化排查树。

## 关键概念
- 思维链 (CoT): 让大模型逐步推理并展示思考过程的方法
- 排查树: 从根节点问题出发，分支展示各种可能排查步骤的树状图
- 元认知分析: 模型对问题信息进行分析和评估的思考环节

## 关联笔记
- 01KJBZZYFRDE3MNYM2SP16GZ3T.md: ChatDBA 代码仓库地址，与本笔记同属 ChatDBA 项目
- 01KJBZYC8BKTFV07JJNAQNS91T.md: 探讨思维链忠实性，与本笔记的 CoT 方案形成理论对照
