---
note: 01KJBZAF5A9DS1CND4J22B9X6V.md
title: 20240627 - 测试通义千问plus和max的差异
indexed_at: 2026-02-27T11:04:33.861621+00:00
---

用户需要我分析这篇关于 MySQL 崩溃排查的笔记，生成索引元数据。让我仔细分析笔记内容：

1. 笔记标题：20240627 - 测试通义千问 plus 和 max 的差异
2. 内容：关于 MySQL 执行 SELECT 语句时崩溃的排查案例，包含诊断代码和对话历史

核心内容：
- MySQL 5.7.36 执行 SELECT 时 crash
- 错误日志显示 signal 8（不是 signal 11）
- 堆栈追踪显示与 decimal2bin、filesort 相关
- AI 辅助诊断流程

标签应该是：MySQL, 崩溃排查，错误日志，signal 8, filesort, decimal2bin

关联笔记分析：
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: gdb 调试 - 关联度低
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 关联度中等
- 01KJBYDAF12RN18BECZG1QSTEC.md: 内存去哪儿了 - 可能相关（内存问题）
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - 关联度低

最相关的应该是 MySQL 相关的笔记。## 标签
MySQL, 崩溃排查, signal 8, 错误日志分析, AI 诊断, decimal2bin

## 摘要
记录使用 AI 辅助诊断 MySQL 5.7.36 执行 SELECT 语句时发生 signal 8 崩溃的排查过程。通过错误日志堆栈追踪定位到 decimal2bin 和 filesort 相关函数，展示多轮对话引导式的诊断方法。

## 关键概念
- signal 8: 浮点异常信号，通常由除零或浮点运算错误触发
- decimal2bin: MySQL 内部函数，负责将十进制数转换为二进制格式
- filesort: MySQL 排序算法，当无法使用索引排序时使用临时文件排序

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（同为 MySQL 内存/崩溃相关问题排查）
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究（同为 MySQL 底层机制研究）
