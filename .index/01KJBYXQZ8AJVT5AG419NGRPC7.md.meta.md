---
note: 01KJBYXQZ8AJVT5AG419NGRPC7.md
title: 20230111 - 农行crash分析整理
indexed_at: 2026-03-05T08:30:24.323063+00:00
---

## 摘要
分析农行 MySQL 8.0.18 两次崩溃（1 月 5 日和 1 月 9 日）的 coredump，发现 temptable 引擎使用 mmap 映射临时文件时发生内存污染，导致行数据指针指向非法地址 0xa0a0a0a0a0a0a0a 引发崩溃。确定两次故障的内存污染范围分别为 896kb 和 64kb，均位于 mmap 映射的临时表文件区域。

## 关键概念
- temptable: MySQL 8.0 内部临时表存储引擎，使用内存或 mmap 文件存储临时数据
- 内存污染: 部分内存区域被异常数据（0xa）覆盖，导致指针访问非法地址
- mmap: 内存映射文件机制，temptable 使用 mmap 将临时文件映射到内存地址空间
- coredump: 程序崩溃时的内存镜像文件，用于 gdb 调试分析崩溃原因

## 关联笔记
- 01KJBYXQRY6V6K04B14ER6EQ02.md: 同一故障系列的早期分析笔记（20230109），包含更详细的 gdb 调试过程
- 01KJBYXQQS6MFV2NVNDXSN404C.md: 同一故障系列的后续分析笔记（20230119），继续验证污染区域跨越结论
- 01KJBYXQAJXVJW2NGMJSS8HCA3.md: 故障分析移交整理笔记（20230128），汇总两次 coredump 分析结果
