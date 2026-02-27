---
note: 01KJBYED7JBE2YBE7Y4YY213H6.md
title: 20210819 - MySQL 内存, 除了p_s统计的, 还有哪些 3 - tcmalloc
indexed_at: 2026-02-27T10:09:17.956090+00:00
---

用户要求我分析一篇关于 MySQL 内存分析的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 主题：MySQL 内存分析，使用 tcmalloc 进行内存 profiling
2. 主要内容：
   - 使用 tcmalloc 观察 MySQL 内存分配
   - 分析了几个内存分配条目（log_allocate_buffer、Link_buf、Pages::add、os_create_block_cache 等）
   - 对比了 os 统计和 tcmalloc 统计的差异

3. 关键概念：
   - tcmalloc 内存分析
   - Link_buf（redo log 并发写入）
   - double write buffer
   - Block cache
   - RSS/VSS 等内存统计

4. 关联笔记：
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（主题相关，都是内存分析）
   - 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释（笔记中涉及 RSS 统计）
   - 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用（都是 MySQL 诊断相关）

现在生成索引：## 标签
MySQL, 内存分析, tcmalloc, redo log, double write buffer, 性能诊断

## 摘要
使用 tcmalloc 对 MySQL 8.0.25 进行内存 Profiling，分析主要内存分配来源（log buffer、Link_buf、double write buffer、block cache 等）。对比 OS 统计（RSS）与 tcmalloc 统计的差异，并通过 smaps 进一步分析内存分布。

## 关键概念
- tcmalloc: Google 开发的线程缓存内存分配器，用于内存 Profiling
- Link_buf: 允许事务并发写入 redo log 的缓冲结构，分 recent_written 和 recent_closed
- double write buffer: InnoDB 防止页断裂的机制，内存消耗为文件大小的两倍
- Block cache: 文件系统块缓存，用于引擎加解密，每块 20.8kB
- RSS: 进程驻留集大小，包含代码段、堆、共享库等所有内存

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同为主题为 MySQL 内存分析的笔记
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 解释了 VSS/RSS/USS/PSS 概念，与笔记中内存统计对比相关
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 也是 MySQL 内存诊断工具之一
