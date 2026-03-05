---
note: 01KJBZ4V4AX7E85YRY3QG1H2W3.md
title: 20240103 - 博般MySQL crash
indexed_at: 2026-03-05T09:26:29.541038+00:00
---

## 摘要
记录 MySQL 5.7.31 两次崩溃的原始工单和堆栈信息，涉及 SHAI-10843 和 SHAI-10993 两个支持工单。崩溃堆栈显示问题集中在 InnoDB 引擎的 trx_undo、btr_cur 和 row_search_mvcc 等底层函数。

## 关键概念
- trx_undo_free_prepared: InnoDB 事务回滚段释放已准备事务的函数，第一次崩溃点
- btr_cur_optimistic_latch_leaves: B+ 树叶节点乐观锁定位函数，两次崩溃均涉及
- btr_pcur_restore_position: 持久化游标恢复位置操作，两次崩溃均出现
- filesort: MySQL 排序操作，堆栈显示崩溃发生在排序查询执行期间

## 关联笔记
- 01KJBZARNH5DADCGZBKQAHK6PD.md: 同为 MySQL 5.7.31 crash coredump 分析，堆栈中出现相同的 btr_cur_optimistic_latch_leaves 和 btr_pcur_restore_position_func 函数
- 01KJBZAQ5MZVR57XF1YJQSGN57.md: ChatDBA 项目使用思维链生成 MySQL crash 排查图的测试笔记
- 01KJBZAPGEQT0E9NB6C8CFG229.md: ChatDBA 项目增量变更排查图方案的后续测试，同样涉及 MySQL select 语句 crash 场景
