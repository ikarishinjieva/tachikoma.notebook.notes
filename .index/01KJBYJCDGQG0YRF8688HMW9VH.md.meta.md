---
note: 01KJBYJCDGQG0YRF8688HMW9VH.md
title: 20220422 - 国债故障 2
indexed_at: 2026-02-27T10:16:22.959933+00:00
---

用户要求我分析这篇关于 MySQL/InnoDB 故障排查的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是关于国债系统故障的排查笔记
2. 主要问题是 SYS_TABLES 的数据页在 buffer pool 中被设置成锁定页，IO 类型为 NONE
3. 涉及 MySQL 5.7.29 版本的 InnoDB 引擎
4. 分析了 buf_fix_count 的加减逻辑
5. 从 SYS_TABLES 的变更触发点进行分析

核心主题：MySQL InnoDB buffer pool、SYS_TABLES、故障排查、fix_count

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 20210406 - innodb 如何后台不刷脏页，前台会怎么处理（与 InnoDB 刷盘相关）
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究（与刷脏页相关）
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿（与刷盘相关）

这些笔记都涉及 MySQL InnoDB 的刷盘/刷脏页机制，与当前笔记的 SYS_TABLES 刷盘问题有关联。

现在生成索引元数据。
## 标签
MySQL, InnoDB, 故障排查, SYS_TABLES, buffer pool, fix_count

## 摘要
分析国债系统中 SYS_TABLES 数据页在 buffer pool 中被异常锁定且 IO 类型为 NONE 的问题。从 fix_count 加减逻辑和 SYS_TABLES 变更触发点两个方向进行代码级排查，发现可能存在缓慢泄露。

## 关键概念
- buf_fix_count: InnoDB 缓冲页的引用计数，用于跟踪页的使用状态
- SYS_TABLES: InnoDB 系统表，存储表元数据信息
- flush_list: InnoDB 刷脏页链表，LSN 推进受其影响
- buffer pool: InnoDB 缓冲池，缓存数据页和索引页
- dict_load_table_low: 加载表定义的函数，涉及 SYS_TABLES 扫描

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 同属 InnoDB 刷脏页机制研究，涉及后台/前台刷盘行为
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究，与 SYS_TABLES 刷盘滞留问题直接相关
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究，涉及数据文件刷盘慢导致卡顿的场景
