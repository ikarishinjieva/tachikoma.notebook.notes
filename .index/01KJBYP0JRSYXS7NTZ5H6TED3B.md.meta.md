---
note: 01KJBYP0JRSYXS7NTZ5H6TED3B.md
title: 20221008 - OB文档阅读
indexed_at: 2026-03-05T08:09:40.155805+00:00
---

## 摘要
记录阅读 OceanBase 企业版官方文档时的问题和思考，涵盖路由策略、副本合并、内存管理等核心机制。TODO 部分列出待深入研究的方向，包括转储/合并机制、内存统计、参数配置等运维主题。

## 关键概念
- OBProxy: OceanBase 数据库代理层，统一路由更容易理解
- 合并 (Major Compaction): 将多版本数据合并，可能引起明显阻塞
- 转储 (Minor Compaction): MemStore 数据写入磁盘的过程
- Route Policy: observer 层的路由策略机制
- Zone: OceanBase 集群的逻辑分区单元

## 关联笔记
- 01KJBYXQTFVXZDQBG4A4V8EBCQ.md: 涉及 observer 启动问题和合并状态查询 (`merge_status`)
- 01KJBZ4FXPMAFNKRB7WKZED2CX.md: observer 编译和启动流程，与文档阅读中的 observer 层机制相关
- 01KJBZ2TD63SRBNM9GPKBXR5G4.md: 涉及 OB 日志流和事务机制，与 TODO 中的数据合并研究相关
