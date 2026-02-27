---
note: 01KJBYXQZ8AJVT5AG419NGRPC7.md
title: 20230111 - 农行crash分析整理
indexed_at: 2026-02-27T10:25:10.479028+00:00
---

用户要求我分析一篇关于农行 MySQL crash 分析的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是关于 2023 年 1 月 5 日和 1 月 9 日两份 coredump 的分析
2. 主要分析的是 temptable（临时表）导致的 MySQL 崩溃
3. 崩溃原因是访问 temptable 表数据时，列数据指向非法地址 0xa0a0a0a0a0a0a0a
4. 通过 gdb 调试定位到问题行在 temptable 的第 7 页第 1510 行

现在需要生成：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个
4. 关联笔记：从列表中找 0-3 篇相关的

看关联笔记列表，这篇笔记是关于 MySQL temptable crash 分析的，查看是否有相关的：
- 01KJBYD9T2SKQ815RG0XWSD236.md: 20210330 - 农行 JDK 导致 mysql connector 报错 - 也是农行的问题，但主题是 connector 报错
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 20210525 - 临时磁盘表，INFORMATION_SCHEMA.FILES 和 performance_schema.file_summary_by_instance 观测不一致 - 这个涉及临时表，有关联


- 01KJBYDCR25528Q2TPMH0SAR9P.md: 20210526 - 农行 C# 程序 SSL 超时 - 也是农行，但主题不相关

所以最相关的是临时磁盘表那篇，因为都涉及临时表问题。农行那篇虽然主题不同，但都是农行的案例，可以算弱关联。
## 标签
MySQL, coredump 分析，temptable, 内存崩溃，gdb 调试，农行

## 摘要
分析 2023 年 1 月 5 日农行 MySQL 崩溃的 coredump，定位到 temptable 临时表访问时因列数据指向非法内存地址导致 memcpy 失败。通过 gdb 遍历 temptable 页和行，找到问题行位于第 7 页第 1510 行，前后行数据正常但该行及后续多行列数据均指向 0xa0a0a0a0a0a0a0a 非法地址。

## 关键概念
- temptable::Handler: MySQL 临时表处理器，负责临时表的读写操作
- coredump: 程序崩溃时的内存快照，用于事后分析崩溃原因
- 迭代器遍历: 通过 m_rnd_iterator 遍历临时表行数据定位问题
- 非法内存访问: 列数据的 m_data 指针指向无效地址导致崩溃

## 关联笔记
- 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 同涉及临时磁盘表相关问题，主题关联
- 01KJBYD9T2SKQ815RG0XWSD236.md: 同为农行 MySQL 问题分析案例
