---
note: 01KJBYDV2NWRKK7EW365CFMZG6.md
title: 20210724 - MySQL 内存, 除了p_s统计的, 还有哪些
indexed_at: 2026-02-27T10:07:29.903291+00:00
---

用户要求我分析一篇关于 MySQL 内存调试的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：使用 gdb 调试 MySQL 内存分配，除了 performance_schema 统计的内存外，还有哪些内存分配
2. **核心内容**：
   - 使用 gdb 监控 malloc 和 pfs_memory_alloc_vc 两个断点
   - 通过抽样方式避免性能影响
   - 分析了 malloc 与 p_s 匹配的调用栈
3. **关键概念**：gdb、performance_schema、malloc、内存分配、调用栈
4. **关联笔记**：
   - 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用（直接相关，都涉及 p_s）
   - 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（主题相关，都是内存分析）
   - 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 20210404 - gdb 只停止触发断点的线程（gdb 技术相关）

现在按照格式输出。## 标签
MySQL, 内存分析, gdb, performance_schema, 调用栈，内存分配

## 摘要
本文介绍如何使用 gdb 建立 MySQL 内存观测方法，通过监控 malloc 和 pfs_memory_alloc_vc 两个断点来追踪内存分配。使用抽样方式避免对性能影响过大，并展示了如何分析 gdb 输出的调用栈信息。

## 关键概念
- gdb 断点抽样：使用 ignore 语句减少断点触发频率，降低性能影响
- pfs_memory_alloc_vc：performance_schema 的内存统计入口函数
- 调用栈分析：通过比对 malloc 与 p_s 的调用栈识别未被 p_s 统计的内存分配

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 同涉及 performance_schema 的内存统计功能
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同属 MySQL 内存分析主题
- 01KJBYD9RFGR2SQZQX5P6CB2HJ.md: 同涉及 gdb 断点调试技术
