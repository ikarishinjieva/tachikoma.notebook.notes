---
note: 01KJBYPH9KR9B4KT123DR6PJAF.md
title: 20221019 - clickhouse "table is in readonly mode" 诊断
indexed_at: 2026-03-05T08:10:15.249805+00:00
---

## 标签
ClickHouse, ZooKeeper, Session Expired, 只读模式, 故障诊断, ReplicatedMergeTree

## 摘要
分析 ClickHouse 表进入只读模式的根本原因，确认为 ZooKeeper 会话超时导致。日志显示 Session expired 错误呈周期性（每 10 分钟）出现，故障持续约 5 小时后因网络/ZK 环境恢复而自动修复。

## 关键概念
- Session Expired: ZooKeeper 客户端与会话服务器之间的连接超时，导致会话失效
- ReplicatedMergeTree: ClickHouse 复制表引擎，依赖 ZooKeeper 协调数据复制
- BackgroundProcessingPool: ClickHouse 后台处理线程池，错误时会指数退避睡眠（最大 600s）

## 关联笔记
- 01KJBYXQWQNYSHP23Q9B3MDJCR.md: 同一光大 ClickHouse 集群的后续问题诊断，涉及 ReplicatedMergeTree 复制队列和 ZooKeeper 交互
- 01KJBYXQPQ1SX1N9ZQFRYPFKGG.md: 同一集群的早期诊断笔记，记录 Too many parts 报错和 ZooKeeper 配置
