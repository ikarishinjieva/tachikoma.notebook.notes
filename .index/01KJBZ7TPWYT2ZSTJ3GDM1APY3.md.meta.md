---
note: 01KJBZ7TPWYT2ZSTJ3GDM1APY3.md
title: 20240402 - 解决使用Plan的问题: 合适进行Plan重新生成
indexed_at: 2026-03-05T09:42:38.391083+00:00
---

## 摘要
记录判断输入是否偏离 Plan 的尝试，通过构建 DBA 专家 AI 在文档库中搜索参考资料进行问题诊断。包含多个 MySQL 故障处理案例（IO 瓶颈、锁等待、死锁、主从同步等）的参考文档片段。

## 关键概念
- Plan 重新生成: 判断当前输入是否偏离 Plan 轨道，决定是否重新生成执行计划
- DBA 专家 AI: 基于 RAG 的数据库专家系统，在文档库中搜索参考资料后回答问题
- innodb_lock_waits: MySQL 系统表，用于查询 InnoDB 行锁阻塞情况
- 死锁日志局限性: 不显示事务已持有的所有锁，仅记录正在执行的 SQL

## 关联笔记
- 01KJBZED41K5CMAYSC5QKC2E76.md: 分析 MySQL mutex 锁的 signal_count 机制，与锁等待主题相关
- 01KJBZA6T5H1DQDV560RRNDGK7.md: ChatDBA 判断参考材料与问题相关性的方法，与 RAG 检索策略相关
- 01KJBZAD3XNGKCT09G5YRWR7WP.md: 测试 GENREAD 通过 LLM 生成召回文档，与文档检索生成相关
