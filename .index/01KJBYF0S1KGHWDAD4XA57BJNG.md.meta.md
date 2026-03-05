---
note: 01KJBYF0S1KGHWDAD4XA57BJNG.md
title: 20210929 - 测试 jemalloc
indexed_at: 2026-03-05T07:45:57.766925+00:00
---

## 摘要
对比 jemalloc 和 tcmalloc 在 MySQL 内存分析中的行为差异，发现 jemalloc 无法统计 mmap 分配的内存量（mmap 不经过 jemalloc），而 tcmalloc 会转发 mmap 调用。jemalloc 开启统计功能时性能与 glibc 几乎一致，无明显衰减。

## 关键概念
- lg_prof_interval: jemalloc 参数，控制每 2^n 字节分配进行一次 dump
- jeprof: jemalloc 的性能分析工具，用于生成 heap profile 的 SVG 可视化
- mmap 拦截: tcmalloc 会拦截 mmap 调用并统计，而 jemalloc 不会
- prof:true: jemalloc 的内存分析开关配置参数

## 关联笔记
- 01KJBYF1AM3RKA6R3JM57VXRYC.md: 深入分析 tcmalloc 性能问题的原因及与 jemalloc 的对比
- 01KJBYF1Q3J13H7W6JV7FNVXN3.md: 介绍如何调试 jemalloc 对 unwind 库的调用方法
- 01KJBYEYVZXZ4D0GEY80NHD3E0.md: 对比 tcmalloc 不同 so 包（minimal 与完整版）的性能差异
