---
note: 01KJBYEX76SJWVW62660M31PCG.md
title: 20210907 - MySQL内存分配的流程
indexed_at: 2026-03-05T07:44:04.546607+00:00
---

## 摘要
笔记描述了 MySQL 向分配器申请内存的四种情况（经过/不经过 performance_schema，使用 malloc/new 或 mmap）。解释了 tcmalloc 如何统计 MySQL 内存申请，以及 HEAP_PROFILE_MMAP 参数对 tcmalloc 统计范围的影响。

## 关键概念
- performance_schema: MySQL 性能监控框架，可选择是否经过它进行内存分配追踪
- tcmalloc: 线程缓存内存分配器，默认统计 malloc/new 申请的内存，会维持一定余量不立即释放给操作系统
- mmap: 内存映射系统调用，用于直接向操作系统申请内存，不经过分配器管理
- 虚拟内存: 操作系统分配的内存是虚拟的，内存被使用时才真正被占用

## 关联笔记
- 01KJBYEB8E2QHWG1X3JVN4V0M1.md: 介绍使用 libtcmalloc 追踪内存分配和生成内存火焰图的方法
- 01KJBYF1Q3J13H7W6JV7FNVXN3.md: 介绍使用 gdb 调试 tcmalloc 的方法及 HEAP_PROFILE_MMAP 参数配置
