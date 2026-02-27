---
note: 01KJBYECA7QEGNNAHJQ0PZ8QPE.md
title: 20210819 - MySQL 内存, 除了p_s统计的, 还有哪些 2
indexed_at: 2026-02-27T10:09:09.511793+00:00
---

用户要求我分析一篇关于 MySQL 内存分析的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 MySQL 内存分析的技术笔记
2. 主要内容是使用 bcc 工具改造 memleak 脚本，加入对 pfs_memory_alloc_vc 的观察
3. 目的是区分被 performance_schema 统计的内存和未被统计的内存
4. 有前序笔记链接

现在我来生成索引：

**标签**：需要 3-6 个关键词
- MySQL
- 内存分析
- bcc
- performance_schema
- 内存泄漏
- 性能调试

**摘要**：2-3 句话
- 介绍如何使用 bcc 的 memleak 脚本改造来追踪 MySQL 内存分配
- 通过将 malloc 与 pfs_memory_alloc_vc 观察相互抵消，识别未被 performance_schema 统计的内存部分

**关键概念**：3-5 个
- bcc: Linux 内核追踪工具框架
- memleak: bcc 提供的内存泄漏追踪脚本
- pfs_memory_alloc_vc: MySQL performance_schema 的内存分配接口
- performance_schema: MySQL 性能监控组件

**关联笔记**：从列表中寻找相关的
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（内存分析主题相关）


- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用（直接涉及 performance_schema）
- 笔记开头提到的前序笔记 [20210724 - MySQL 内存，除了 p_s 统计的，还有哪些] 不在给定列表中## 标签
MySQL, 内存分析, bcc, performance_schema, 内存追踪, 性能调试

## 摘要
介绍如何使用 bcc 工具改造 memleak 脚本，加入对 MySQL 的 pfs_memory_alloc_vc 接口追踪。通过将 malloc 与 pfs_memory_alloc_vc 的观察相互抵消，识别未被 performance_schema 统计的内存分配部分。

## 关键概念
- bcc: Linux 内核追踪工具框架，支持 eBPF 技术
- memleak: bcc 提供的内存泄漏追踪脚本，可监控 malloc/free 调用
- pfs_memory_alloc_vc: MySQL performance_schema 的内存分配接口
- performance_schema: MySQL 内置性能监控组件，统计各类资源使用

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同属 MySQL 内存分析主题，探索内存去向问题
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 涉及 performance_schema 的日常使用，与本文追踪对象相关
