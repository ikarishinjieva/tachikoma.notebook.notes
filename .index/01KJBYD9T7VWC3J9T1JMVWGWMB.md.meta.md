---
note: 01KJBYD9T7VWC3J9T1JMVWGWMB.md
title: 20210330 - performance_schema的日常使用
indexed_at: 2026-02-27T10:01:05.534065+00:00
---

用户要求我分析一篇关于 MySQL performance_schema 的笔记，并生成索引元数据。让我分析笔记内容：

笔记标题：20210330 - performance_schema 的日常使用

笔记内容：
```
call sys.ps_setup_enable_instrument('');
call sys.ps_setup_enable_consumer('');
call sys.ps_setup_disable_background_threads();
```

这是关于 MySQL performance_schema 的配置命令，使用 sys 库的存储过程来启用/禁用性能监控功能。

从其他笔记列表中，我需要找出相关的笔记：
- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错 - 同一天，MySQL 相关
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 - MySQL 性能相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 - MySQL 性能相关
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 20210525 - 临时磁盘表，INFORMATION_SCHEMA.FILES 和 `performance_schema`.`file_summary_by_instance` 观测不一致 - 直接提到 performance_schema

最相关的是 01KJBYDAZJTQDJ7X3BQ52XVSSR.md，因为它直接提到了 performance_schema。

现在生成索引：

标签：MySQL, performance_schema, 性能监控，sys 库，调试


摘要：记录 MySQL performance_schema 的日常配置方法，通过 sys 库的存储过程启用监控工具和消费者，同时可禁用后台线程以减少干扰。

关键概念：
1. performance_schema: MySQL 内置的性能监控框架
2. sys.ps_setup_enable_instrument: 启用性能监控工具的存储过程
3. sys.ps_setup_enable_consumer: 启用性能数据消费者的存储过程
4. sys.ps_setup_disable_background_threads: 禁用后台线程监控的存储过程

关联笔记：
01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 同样涉及 performance_schema 的使用场景## 标签
MySQL, performance_schema, 性能监控，sys 库，调试

## 摘要
记录 MySQL performance_schema 的日常配置命令。通过 sys 库的存储过程启用监控工具和数据消费者，并可禁用后台线程以减少干扰。

## 关键概念
- performance_schema: MySQL 内置的性能监控框架，用于收集数据库运行时性能数据
- sys.ps_setup_enable_instrument: 启用指定性能监控工具的存储过程
- sys.ps_setup_enable_consumer: 启用性能数据消费者的存储过程
- sys.ps_setup_disable_background_threads: 禁用后台线程监控以避免干扰分析

## 关联笔记
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 同样涉及 performance_schema 的使用和观测
