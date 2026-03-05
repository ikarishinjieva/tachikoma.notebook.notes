---
note: 01KJBYEY2E83QAP1T2T8KJ5HA2.md
title: 20210923 - hash join 使用文件句柄的数量
indexed_at: 2026-03-05T07:45:25.871502+00:00
---

## 摘要
记录 MySQL Hash Join 实现机制（Hybrid Hash Join）及其产生大量文件句柄的问题。通过 perf 工具追踪 sys_exit_openat 系统调用，分析文件句柄消耗与 chunk 切分机制的关联。

## 关键概念
- Hybrid Hash Join: MySQL 的 Hash Join 实现方式，将表切分为 chunk 放在磁盘中进行内存哈希连接
- build table: Hash Join 中用于构建哈希表的表，被切分为 128 个 chunk 文件
- probe table: Hash Join 中用于探测哈希表的表，同样被切分为 128 个 chunk 文件
- perf: Linux 性能分析工具，用于追踪系统调用和生成调用栈

## 关联笔记
- 01KJBYF0S1KGHWDAD4XA57BJNG.md: 同样使用 perf 追踪 MySQL 系统调用（sys_enter_mmap）进行内存分配分析
- 01KJBZ9A3SBB26TS11D29F5C93.md: 讨论 hash join 处理大量数据时产生巨大临时文件的问题
