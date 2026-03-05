---
note: 01KJBYFBDSR1SNE0X711Z0TGXR.md
title: 20211117 - retdec 会导致SIGSEGV的探索
indexed_at: 2026-03-05T07:50:23.317719+00:00
---

## 标签
retdec, anvill, 反编译, LLVM IR, SIGSEGV, 二进制分析

## 摘要
在尝试 mcsema/reopt 等工具失败后，重新探索 retdec 反编译工具。对比 retdec 和 anvill 生成的 LLVM IR 文件，发现 anvill 输出的 ll 文件更长且可读性更差。

## 关键概念
- retdec: 可重定向的反编译器，将二进制代码转换为 LLVM IR
- anvill: 基于 IDA Pro 的反编译工具，生成 LLVM IR
- LLVM IR: LLVM 中间表示，用于代码优化和分析
- mcsema: 二进制代码提升工具，将 x86 指令转换为 LLVM IR

## 关联笔记
- 01KJBYF33K90HXENWZZKHBND30.md: retdec 的早期试用记录，记录了 retdec 的目的和使用过程
- 01KJBYFAAKD1C10V96SPYJTY6A.md: mcsema 和 anvill 的编译安装及使用尝试，与本笔记的工具链直接相关
- 01KJBYF973H98W1RNZ05SS50KE.md: 记录 retdec 生成的 LL 文件编译后出现 SIGSEGV 问题的后续调查
