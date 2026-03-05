---
note: 01KJBZ7SQMZGYSPHGCPSH9BVB6.md
title: 20240329 - ChatDBA在提示词中使用plan
indexed_at: 2026-03-05T09:41:43.982847+00:00
---

## 标签
ChatDBA, 提示词工程, MySQL 故障处理，锁等待，死锁，IO 瓶颈，慢查询

## 摘要
记录 ChatDBA 提示词中参考文档的结构设计，包含多个 MySQL 故障处理案例的工单复盘。涵盖锁等待、死锁、IO 资源不足、存储引擎选择等典型问题的诊断思路和解决方案。

## 关键概念
- 死锁日志: InnoDB 记录死锁发生时间、事务信息及持有/等待的锁，但存在不显示所有持有锁和 SQL 的局限性
- innodb_lock_wait_timeout: InnoDB 锁等待超时参数，默认值较短，行锁阻塞通常自动解除
- 存储引擎锁机制: MyISAM 使用表锁，InnoDB 使用行锁，DML 频繁场景建议使用 InnoDB
- 阻塞源定位: 通过 sys.innodb_lock_waits 统计阻塞其他线程最多的连接来识别阻塞源

## 关联笔记
- 01KJBZED41K5CMAYSC5QKC2E76.md: 分析 MySQL mutex 锁的 signal_count 机制，与锁等待主题相关
- 01KJBZB3ESK010EVRSR0XJDJFA.md: ChatDBA 使用思维链进行问题排查的训练方法，与提示词设计相关
