---
note: 01KJBYEX76SJWVW62660M31PCG.md
title: 20210907 - MySQL内存分配的流程
indexed_at: 2026-02-27T10:10:25.059542+00:00
---

用户要求我分析一篇关于 MySQL 内存分配的笔记，并生成索引元数据。让我仔细阅读笔记内容，然后按照指定格式输出。

笔记核心内容：
1. MySQL 向分配器申请内存的流程（经过/不经过 performance_schema，malloc/new 或 mmap）
2. 分配器向操作系统申请内存（tcmalloc 的行为）
3. 操作系统分配虚拟内存的机制

标签应该包括：MySQL、内存分配、performance_schema、tcmalloc、malloc、mmap 等

摘要需要概括核心内容

关键概念：performance_schema、malloc/new、mmap、tcmalloc、虚拟内存

关联笔记：从列表中找相关的
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了（内存相关）
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 20210330 - performance_schema 的日常使用（performance_schema 相关）
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 20210628 - VSS/RSS/USS/PSS 解释（内存相关）

让我按照格式输出。## 标签
MySQL, 内存分配, performance_schema, tcmalloc, malloc, mmap

## 摘要
介绍 MySQL 向分配器申请内存的四种场景（经过/不经过 performance_schema × malloc/mmap）。说明 tcmalloc 对 malloc/new 和 mmap 内存的统计机制，以及操作系统虚拟内存的分配原理。

## 关键概念
- performance_schema: MySQL 性能监控组件，可追踪内存分配路径
- malloc/new: 需要分配器管理的内存申请方式
- mmap: 直接向操作系统申请内存，绕过分配器管理
- tcmalloc: 线程缓存分配器，可维持内存余量不立即释放给操作系统
- 虚拟内存: 操作系统分配的内存，实际使用时才真正占用

## 关联笔记
- 01KJBYDAF12RN18BECZG1QSTEC.md: 同为 MySQL 内存分析主题，探讨内存去向
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 涉及 performance_schema 的日常使用
- 01KJBYDNQ4TQXJPQCVEH94QW3E.md: 解释 VSS/RSS/USS/PSS 等内存指标，与虚拟内存概念相关
