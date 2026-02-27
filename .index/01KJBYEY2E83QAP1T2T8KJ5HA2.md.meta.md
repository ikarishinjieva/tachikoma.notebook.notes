---
note: 01KJBYEY2E83QAP1T2T8KJ5HA2.md
title: 20210923 - hash join 使用文件句柄的数量
indexed_at: 2026-02-27T10:11:11.738028+00:00
---

用户希望我分析这篇关于 MySQL Hash Join 使用文件句柄的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **标签**：需要 3-6 个关键词
   - MySQL
   - Hash Join
   - 文件句柄
   - Hybrid Hash Join
   - 性能调试
   - perf

2. **摘要**：2-3 句话概括核心内容
   - 笔记研究了 MySQL Hash Join 实现机制及其文件句柄使用问题
   - 通过 perf 工具追踪文件句柄的创建来源

3. **关键概念**：3-5 个核心概念
   - Hybrid Hash Join: MySQL 的哈希连接实现方式
   - Build/Probe Table: Hash Join 中的两张表角色
   - Chunk: 表切分的数据块
   - 文件句柄: 系统打开文件的描述符
   - perf: Linux 性能分析工具

4. **关联笔记**：从列表中找相关的
   - 01KJBYDBWTJGYAJWBZ93943E5R.md: 20210531 - Clickhouse Join 研究（都是关于 Join 的研究）
   - 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: 20210622 - Clickhouse distributed join 学习（也是 Join 相关）
   - 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用（都是性能观测相关）

现在按照指定格式输出。
## 标签
MySQL, Hash Join, 文件句柄, Hybrid Hash Join, perf, 性能调试

## 摘要
笔记研究 MySQL Hash Join 的实现机制，其采用 Hybrid Hash Join 方式将表切分为 chunk 存储在磁盘中。通过 perf 工具追踪 Hash Join 执行时产生大量文件句柄的根源。

## 关键概念
- Hybrid Hash Join: MySQL 将 join 表切分为 chunk 放在磁盘，部分加载内存进行哈希连接的实现方式
- Build/Probe Table: Hash Join 中两张表的角色划分，分别加载进内存进行连接
- Chunk: 表切分的数据块，每种 chunk table 有 128 个文件
- 文件句柄: 进程打开文件的描述符，Hash Join 执行时会消耗大量文件句柄
- perf: Linux 性能分析工具，用于追踪 sys_exit_openat 系统调用定位文件打开来源

## 关联笔记
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 同为 Join 技术研究方向，Clickhouse Join 研究可对比参考
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 同为 MySQL 性能观测工具使用，performance_schema 与 perf 可互补
- 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: 同为分布式 Join 研究，Clickhouse distributed join 可对比 MySQL 实现
