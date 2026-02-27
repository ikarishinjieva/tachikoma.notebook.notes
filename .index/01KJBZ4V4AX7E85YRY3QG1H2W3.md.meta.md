---
note: 01KJBZ4V4AX7E85YRY3QG1H2W3.md
title: 20240103 - 博般MySQL crash
indexed_at: 2026-02-27T10:41:54.948415+00:00
---

用户要求我分析这篇关于 MySQL crash 的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：MySQL 5.7.31 崩溃问题分析
2. **核心内容**：记录了两次 MySQL 崩溃的堆栈信息，涉及 InnoDB 存储引擎
3. **关键信息**：
   - 数据库版本：MySQL 5.7.31
   - 崩溃两次，有工单链接
   - 堆栈显示涉及 innodb 的 trx_undo、btr_cur、btr_pcur、row_search_mvcc 等函数
   - 第二次崩溃涉及 dtuple_convert_big_rec、BtrBulk::insert 等

4. **可能的标签**：MySQL、崩溃分析、InnoDB、堆栈跟踪、故障排查

5. **关联笔记分析**：
   - 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 后台刷脏页相关 - 有一定关联（都是 InnoDB）
   - 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 有关联（InnoDB 相关）
   - 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 临时磁盘表相关 - 关联度低
   - 01KJBYDMS7X3DPNYC2CZBTRV2F.md: Adaptive hash index 代码解析 - 都是 MySQL 内部机制研究

最相关的应该是 InnoDB 相关的笔记，特别是刷脏页研究的。

现在我来生成索引元数据。
## 标签
MySQL, 崩溃分析, InnoDB, 堆栈跟踪, 故障排查

## 摘要
记录 MySQL 5.7.31 两次崩溃的工单及堆栈信息。崩溃点集中在 InnoDB 的 B-Tree 操作（btr_cur_optimistic_latch_leaves、btr_pcur_restore_position）和事务回滚段（trx_undo_free_prepared）相关函数。

## 关键概念
- trx_undo_free_prepared: InnoDB 事务回滚段清理函数
- btr_cur_optimistic_latch_leaves: B-Tree 叶节点乐观锁定位
- btr_pcur_restore_position: 持久化游标位置恢复
- row_search_mvcc: InnoDB MVCC 行搜索函数
- dtuple_convert_big_rec: 大记录格式转换函数

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同为 InnoDB 后台刷页机制研究
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 同为 InnoDB 刷脏页相关研究
- 01KJBYDMS7X3DPNYC2CZBTRV2F.md: 同为 MySQL InnoDB 内部机制代码解析
