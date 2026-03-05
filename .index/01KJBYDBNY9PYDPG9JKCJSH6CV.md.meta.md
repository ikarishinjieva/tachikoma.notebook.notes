---
note: 01KJBYDBNY9PYDPG9JKCJSH6CV.md
title: 20210428 - druid连接池 连接维护逻辑整理
indexed_at: 2026-03-05T07:27:06.496715+00:00
---

## 摘要
整理 Druid 连接池的连接维护逻辑，包括 CreatorThread 创建连接、DestroyThread 关闭连接的触发条件和检查项。涵盖心跳检查机制、获取/归还连接时的验证策略，以及配置参数间的关联建议。

## 关键概念
- CreatorThread: 负责创建连接的线程，根据 keepalive、minIdle、maxActive 等条件决定是否建立新连接
- DestroyThread: 定期执行连接回收检查的线程，按 timeBetweenEvictionRunsMillis 间隔触发
- 心跳检查: 通过 COM_PING 或 validationQuery 检测连接有效性，支持 testOnBorrow/testWhileIdle/testOnReturn 三种触发方式
- removeAbandoned: 检测连接泄露的配置，超过 removeAbandonedTimeoutMillis 且无 SQL 运行的连接会被强制关闭

## 关联笔记
- 01KJBZ83MN04S8N329FM6BMDXB.md: 连接池空闲超时与 MySQL wait_timeout 配置不匹配导致的连接异常案例分析
- 01KJBZ4E6HJKFS7FGETVFA76RD.md: 包含 Druid 连接池实际运行日志和 CreateConnectionThread/DestroyConnectionThread 堆栈信息
