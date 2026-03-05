---
note: 01KJBYJCDGQG0YRF8688HMW9VH.md
title: 20220422 - 国债故障 2
indexed_at: 2026-03-05T07:58:13.593497+00:00
---

## 摘要
分析 SYS_TABLES 数据页在 buffer pool 中被设置为锁定页且 IO 类型为 NONE 的问题。通过代码分析追踪 buf_fix_count 的加减逻辑，发现可能存在 fix_count 不对称导致的页滞留问题。

## 关键概念
- buf_fix_count: InnoDB buffer pool 中页面的引用计数，用于追踪页面被固定的次数
- SYS_TABLES: InnoDB 系统表，存储表元数据信息
- flush_list: InnoDB 刷脏页列表，受 SYS_TABLES 滞留影响但 redo log LSN 正常推进
- buf_block_fix: 增加页面 fix_count 的函数，调用链复杂

## 关联笔记
- 01KJBYKCTPB0T5X02RSRVRS3W4.md: 同样涉及 buf_fix_count 断言失败的 MySQL crash 问题分析
- 01KJBZ9A3SBB26TS11D29F5C93.md: 国债生产环境临时文件不释放问题，同一故障场景
