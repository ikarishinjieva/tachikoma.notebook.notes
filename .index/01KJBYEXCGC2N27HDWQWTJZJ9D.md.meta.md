---
note: 01KJBYEXCGC2N27HDWQWTJZJ9D.md
title: 20210915 - 编译gperftools / tcmalloc
indexed_at: 2026-03-05T07:44:28.775267+00:00
---

## 摘要
记录在 CentOS 7 虚拟机上编译 gperftools/tcmalloc 2.9.1 并打包成 RPM 的完整流程。包括安装 rpm-build、下载源码、修改 rpm.spec 文件、执行打包脚本等步骤。

## 关键概念
- gperftools: Google 性能工具套件，包含 tcmalloc 内存分配器和性能分析器
- tcmalloc: Thread-Caching Malloc，高性能多线程内存分配器
- RPM spec: RPM 包的构建规范文件，定义编译和打包流程
- rpm-build: CentOS 下用于构建 RPM 包的工具集

## 关联笔记
- 01KJBYEX76SJWVW62660M31PCG.md: 讲解 tcmalloc 内存统计原理，使用同一 gperftools 2.9.1 版本
- 01KJBYEYVZXZ4D0GEY80NHD3E0.md: 分析 tcmalloc 不同 so 包性能差异，基于此编译的 gperftools 2.9.1
- 01KJBYF1AM3RKA6R3JM57VXRYC.md: 探究 tcmalloc 性能问题，使用相同编译环境和版本
