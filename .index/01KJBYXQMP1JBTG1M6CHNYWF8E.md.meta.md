---
note: 01KJBYXQMP1JBTG1M6CHNYWF8E.md
title: 20221210 - 光大CK 调大后台线程数导致内存急速增长
indexed_at: 2026-03-05T08:24:32.331000+00:00
---

## 摘要
记录了调大 ClickHouse 后台线程数 (background_pool_size 从 16 到 32) 后内存急速上涨的问题。日志分析发现 9 个重点线程在执行 mutation 任务时共消耗约 187GiB 内存，其中线程 80 加载 14187 条 mutation 条目单独消耗 27GiB。结论指出 background_pool_size 需选择合适值，平衡后台线程充足与 mutation 并发过大导致的内存消耗。

## 关键概念
- background_pool_size: ClickHouse 后台线程池大小参数，控制并发后台任务数
- Mutation: ClickHouse 的数据变更操作 (DELETE/UPDATE)，以异步队列方式执行
- ReplicatedMergeTreeQueue: 复制表的队列管理组件，负责加载和执行 mutation 条目
- MemoryTracker: ClickHouse 内存追踪工具，用于分析各线程内存占用

## 关联笔记
- 01KJBYVTTVNYTSRBX5JBT2MJKP.md: 前继笔记，分析光大 AST is too big 报错问题，同样涉及 background_pool_size 参数调优
- 01KJBYXQWQNYSHP23Q9B3MDJCR.md: 同一问题系列的早期笔记，记录 AST is too big 报错的日志分析
- 01KJBYWMG762P36NB4CPT3P64J.md: 光大 CK 同一时期的 DELETE 数据失败问题，涉及相同的表和集群
