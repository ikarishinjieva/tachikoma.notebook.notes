---
note: 01KJBYF973H98W1RNZ05SS50KE.md
title: 20211109 - opt4db 实现
indexed_at: 2026-03-05T07:49:26.066768+00:00
---

## 摘要
记录 opt4db 项目实现过程中遇到的 LLVM IR 生成问题。retdec 生成的 IR 编译时报 SEGSIGV，reopt 可正确生成但会丢失部分函数，llvm-mctoll 解析 MySQL 二进制文件时因未知 opcode 断言失败。

## 关键概念
- opt4db: 基于 dynimize 原理实现的优化项目
- reopt: 用于生成 LLVM IR 的工具，但存在函数丢失问题
- llvm-mctoll: LLVM 反汇编工具，遇到未知 X86 opcode 会断言失败
- LLVM IR: LLVM 中间表示，用于编译优化和反编译

## 关联笔记
- 01KJBYFBDSR1SNE0X711Z0TGXR.md: 后续探索 retdec 导致 SIGSEGV 问题的详细分析
