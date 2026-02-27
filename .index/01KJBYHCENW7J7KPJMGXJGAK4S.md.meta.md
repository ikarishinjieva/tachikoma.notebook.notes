---
note: 01KJBYHCENW7J7KPJMGXJGAK4S.md
title: 20220302 - 工行MySQL crash
indexed_at: 2026-02-27T10:16:02.147781+00:00
---

用户希望我分析这篇关于工商银行 MySQL crash 的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 这是一个关于工行 MySQL 崩溃的分析笔记
2. 涉及工单 BEIJ-2241
3. 包含崩溃堆栈分析
4. MySQL 是工行独自编译，运行在信创 OS 上
5. 分析了堆栈断点问题，怀疑是编译参数优化导致堆栈信息丢失
6. 提供了可复现的断点堆栈
7. 崩溃信号为 signal 11（段错误）

现在我需要生成：
1. 标签：3-6 个关键词
2. 摘要：2-3 句话
3. 关键概念：3-5 个
4. 关联笔记：0-3 篇

关联笔记分析：
- 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 后台刷脏页 - 有一定关联，因为笔记涉及 innodb 的 purge 操作
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 有关联，涉及 innodb 内部机制
- 01KJBYD9T2SKQ815RG0XWSD236.md: 农行 JDK 导致 mysql connector 报错 - 都是银行 MySQL 问题，但技术点不同

最相关的应该是关于 innodb 内部机制的笔记，因为这篇笔记涉及 btr/row/purge 等 innodb 内部操作。## 标签
MySQL, 崩溃分析, 工行, InnoDB, 堆栈调试, 信创

## 摘要
记录工商银行 MySQL 实例崩溃问题的分析过程，崩溃信号为 signal 11。通过堆栈分析定位到 InnoDB 删除操作中的地址错误，怀疑编译优化导致堆栈信息丢失，并提供了可复现的断点堆栈。

## 关键概念
- signal 11: 段错误信号，通常由非法内存地址访问引起
- page_cur_delete_rec: InnoDB 中删除页面记录的核心函数
- btr_cur_optimistic_delete: B+ 树游标的乐观删除操作
- row_purge: InnoDB 清理删除标记记录的后台操作
- 编译优化挤压: 高级别优化可能导致堆栈帧信息丢失

## 关联笔记
- 01KJBYD9S0A3VZWG60QKDTAF49.md: 涉及 InnoDB 后台刷脏页机制，与 purge 操作相关
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究，同属 InnoDB 内部机制分析
- 01KJBYD9T2SKQ815RG0XWSD236.md: 同为银行 MySQL 问题排查笔记
