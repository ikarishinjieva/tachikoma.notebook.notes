---
note: 01KJBYDQEEGP77JKDE6H3659W8.md
title: 20210718 - 组提交性能问题诊断
indexed_at: 2026-03-05T07:35:55.219321+00:00
---

## 标签
MySQL, 半同步复制, binlog, 性能诊断, gdb, 调用栈分析

## 摘要
记录 MySQL 半同步复制场景下的性能问题诊断过程，通过 binlog trace log 定位延迟出现在 ActiveTranx 插入与查询之间。使用 gdb 堆栈分析追踪代码调用路径，涉及 binlog.after_flush 和 Binlog_sender::send_events 两个关键链路。

## 关键概念
- ActiveTranx: MySQL 半同步插件中跟踪事务状态的机制，记录 binlog 位置用于同步确认
- ReplSemiSyncMaster: MySQL 半同步复制主库端插件，负责等待从库 ACK 确认
- binlog.after_flush: binlog 刷新后的回调钩子，用于通知半同步模块事务已写入
- updateSyncHeader: 半同步复制中更新同步位点的函数，在发送 binlog 事件前调用

## 关联笔记
- 01KJBZAQ5MZVR57XF1YJQSGN57.md: 同样涉及半同步复制超时和 binlog 等待问题的排查场景
- 01KJBZEE9JTH4RKQG664GFW88H.md: 使用 gdb 堆栈分析诊断 MySQL crash 问题的类似方法论
- 01KJBZA7W62X0HJ2HGDXSYNQ7Z.md: 主从复制延迟问题的排查思路和常见场景
