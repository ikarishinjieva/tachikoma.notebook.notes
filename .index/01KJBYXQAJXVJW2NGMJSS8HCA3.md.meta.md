---
note: 01KJBYXQAJXVJW2NGMJSS8HCA3.md
title: 20230128 - 农行crash信息整理并移交
indexed_at: 2026-02-27T10:22:07.444061+00:00
---

用户需要我分析这篇关于农行 MySQL crash 的笔记，生成索引元数据。让我分析笔记内容：

1. 这是关于农业银行 MySQL 数据库崩溃问题的调试笔记
2. 涉及两次崩溃（1 月 5 日和 1 月 9 日）
3. 使用 GDB 调试 coredump
4. 分析了问题 SQL
5. 涉及 MySQL、GDB、coredump 等概念

关联笔记分析：
- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错 - 也是农行相关的问题
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用 - MySQL 调试相关
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 20210404 - gdb 只停止触发断点的线程 - GDB 调试相关

现在生成索引：

标签：MySQL, 崩溃分析，GDB, coredump, 农行，SQL 调试

摘要：记录农业银行 MySQL 数据库两次崩溃（1 月 5 日和 1 月 9 日）的问题分析和 coredump 调试过程。通过 GDB 分析获取了导致崩溃的完整 SQL 语句，调试环境已与生产 ARM 环境保持一致。
 
关键概念包括 GDB 调试工具用于分析 coredump 文件、THD 作为 MySQL 线程处理对象、coredump 记录崩溃时内存状态，以及 ARM 作为生产环境的架构。关联笔记中，农行相关的 MySQL connector 报错问题和 GDB 断点调试技术都与本次崩溃分析直接相关。## 标签
MySQL, 崩溃分析，GDB, coredump, 农行，SQL 调试

## 摘要
记录农业银行 MySQL 数据库两次崩溃（1 月 5 日和 1 月 9 日）的问题分析和 coredump 调试过程。通过 GDB 分析获取了导致崩溃的完整 SQL 语句，调试环境已与生产 ARM 环境保持一致。

## 关键概念
- THD: MySQL 线程处理对象，存储连接会话状态和查询信息
- coredump: 程序崩溃时的内存快照，用于事后调试分析
- GDB: GNU 调试器，用于分析 coredump 定位崩溃原因
- ARM 环境: 生产服务器使用的 CPU 架构，调试环境需与之保持一致

## 关联笔记
- 01KJBYD9T2SKQ815RG0XWSD236.md: 同为农行 MySQL 相关问题排查记录
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: GDB 调试技术相关笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: MySQL performance_schema 调试工具使用
