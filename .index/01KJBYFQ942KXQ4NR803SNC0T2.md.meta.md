---
note: 01KJBYFQ942KXQ4NR803SNC0T2.md
title: 20211230 - 国债MySQL 8.0 insert性能与MySQL 5.7有差异
indexed_at: 2026-02-27T10:14:53.836051+00:00
---

用户要求我分析一篇关于 MySQL 8.0 和 5.7 性能差异的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. **主题**：MySQL 8.0 vs 5.7 的 INSERT 性能差异分析
2. **方法**：使用火焰图、perf 等性能分析工具
3. **发现**：8.0 性能低于 5.7，通过多种手段排查（binlog、内存映射、CRC 计算、page fault 等）
4. **结论**：8.0 的 log_buffer_write 调用次数比 5.7 多 20 倍

现在我需要生成：
1. 标签（3-6 个）
2. 摘要（2-3 句话）
3. 关键概念（3-5 个）
4. 关联笔记（0-3 篇）

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 刷脏页研究 - 有关联（都是 InnoDB 性能）
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用 - 有关联（笔记中用到 perf 和 performance_schema）
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 有关联（InnoDB 刷页）
- 01KJBYDAF12RN18BECZG1QSTEC.md: 内存去哪儿了 - 弱关联（涉及内存分析）


- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - 有关联（刷盘性能）

最相关的是 performance_schema 和刷盘/刷脏页相关的笔记。## 标签
MySQL 性能分析，InnoDB，火焰图，版本对比，INSERT 性能，perf

## 摘要
通过火焰图、perf 等工具对比分析 MySQL 8.0 与 5.7 在 INSERT 场景下的性能差异。发现 8.0 的 log_buffer_write 调用次数比 5.7 多 20 倍，且存在更多 page fault，是导致性能下降的主要原因。

## 关键概念
- 火焰图：可视化性能分析工具，用于定位 CPU 热点和调用栈
- page fault：缺页中断，8.0 比 5.7 产生更多内存页故障
- log_buffer_write：InnoDB 日志缓冲区写入函数，8.0 调用频率显著高于 5.7
- perf stat：Linux 性能计数工具，用于统计硬件事件如缓存命中率

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: performance_schema 的日常使用 - 同属 MySQL 性能分析主题
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 - 同属 InnoDB 刷盘性能研究
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 同属 InnoDB 刷页机制研究
