---
note: 01KJBZAEGGE7FQDJ8V3Z2ZGNPH.md
title: 20240702 - ChatDBA: 测试两种数据库的工单混合后的相互影响
indexed_at: 2026-03-05T10:17:11.088073+00:00
---

## 摘要
测试 MySQL 参考材料对 PostgreSQL 排查计划是否会产生错误影响，包含 PostgreSQL 死锁排查的提示词模板和增量修改规范。设计了工单匹配程度评估体系和带_SPECULATIVE 后缀的增量修改函数。

## 关键概念
- 排查计划: AI 生成的数据库问题诊断流程，使用 SWITCH/Case/THINKING/FIND/IF/RETURN 等伪代码函数
- 增量修改: 使用带_SPECULATIVE 后缀的函数对排查计划打补丁，而非输出完整修改后的计划
- 匹配程度: 工单与问题的相关性评级（1-4 级，从完全匹配到完全无关）
- 死锁处理: 数据库并发事务间资源竞争导致的锁定问题，需分析日志和事务执行过程

## 关联笔记
- 01KJBZAHR57R0KFDQ5B0GGG7GT.md: 同一天 ChatDBA 多步推理测试笔记，尝试让 AI 使用查询 Agent 进行逐步思考
