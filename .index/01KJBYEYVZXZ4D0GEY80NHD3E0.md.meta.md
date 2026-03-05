---
note: 01KJBYEYVZXZ4D0GEY80NHD3E0.md
title: 20210928 - tcmalloc 不同so包性能有差异
indexed_at: 2026-03-05T07:45:38.944588+00:00
---

## 摘要
记录 libtcmalloc_minimal.so 与 libtcmalloc.so 在 MySQL 性能测试中表现差异巨大（tps 10624 vs 5.32）。通过对比编译日志发现两者编译选项和链接库不同，最终定位问题是 HEAPPROFILE 参数导致 libtcmalloc.so 性能严重下降。

## 关键概念
- libtcmalloc_minimal: gperftools 的精简版内存分配器，不包含堆检查和性能分析功能
- libtcmalloc: gperftools 的完整版内存分配器，包含 heap-checker 和 heap-profiler 功能
- HEAPPROFILE: tcmalloc 的堆内存分析参数，开启后会严重影响性能
- gperftools: Google 开发的性能工具集，包含 tcmalloc 内存分配器和性能分析工具

## 关联笔记
- 01KJBYF1AM3RKA6R3JM57VXRYC.md: 本笔记结论指向该笔记，详细分析 tcmalloc 性能下降的根本原因
- 01KJBYEX76SJWVW62660M31PCG.md: 描述 MySQL 使用 tcmalloc 进行内存分配的完整流程
- 01KJBYEXCGC2N27HDWQWTJZJ9D.md: 记录 gperftools/tcmalloc 的编译方法，与本笔记的编译日志分析相关
