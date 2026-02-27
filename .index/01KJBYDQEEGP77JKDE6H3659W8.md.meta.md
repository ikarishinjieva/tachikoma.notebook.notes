---
note: 01KJBYDQEEGP77JKDE6H3659W8.md
title: 20210718 - 组提交性能问题诊断
indexed_at: 2026-02-27T10:06:45.544370+00:00
---

用户要求我分析一篇关于 MySQL 组提交性能问题诊断的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 MySQL 半同步复制性能问题的诊断笔记
2. 涉及 binlog trace log 分析
3. 使用 gdb 调试分析了代码调用栈
4. 延迟出现在 ActiveTranx::insert_tranx_node 和 ActiveTranx::is_tranx_end_pos 之间

需要生成的内容：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个
4. 关联笔记：从提供的笔记列表中找出相关的

关联笔记分析：
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: gdb 调试相关 - 有关联
- 01KJBYD9RN1BMGGWMYS78NG817.md: LWP 降速 - 可能有关联（性能调试）
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 刷脏页 - 不太相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 不太相关
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - 可能有关联（都是性能问题）


- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 从 binlog 中分离 SQL - 有关联（都涉及 binlog）

最相关的应该是 gdb 调试那篇和 binlog 那篇。## 标签
MySQL, 半同步复制, binlog, 性能诊断, gdb, 组提交

## 摘要
记录了一次 MySQL 半同步复制延迟问题的诊断过程，通过 binlog trace log 定位延迟出现在 ActiveTranx::insert_tranx_node 和 ActiveTranx::is_tranx_end_pos 之间。使用 gdb 分析代码调用栈，梳理了 binlog 刷新与半同步复制的交互流程。

## 关键概念
- ActiveTranx: MySQL 半同步复制中跟踪活跃事务的数据结构
- binlog.after_flush: binlog 刷新后的回调钩子，用于通知半同步复制模块
- repl_semi_report_binlog_update: 半同步主库在 binlog 更新时调用的回调函数
- MYSQL_BIN_LOG::ordered_commit: MySQL binlog 有序提交的核心函数
- 半同步复制: 主库至少等待一个从库确认收到 binlog 后才返回给客户端

## 关联笔记
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: gdb 只停止触发断点的线程（同为 gdb 调试 MySQL 相关）
- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 从 binlog 中分离 SQL, 进行统计（同涉及 binlog 分析）
