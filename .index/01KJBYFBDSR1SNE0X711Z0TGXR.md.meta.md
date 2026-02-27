---
note: 01KJBYFBDSR1SNE0X711Z0TGXR.md
title: 20211117 - retdec 会导致SIGSEGV的探索
indexed_at: 2026-02-27T10:13:48.820292+00:00
---

The user wants me to analyze a note and generate index metadata in a specific Markdown format. Let me analyze the note:

**Title**: 20211117 - retdec 会导致 SIGSEGV 的探索 (Exploration of retdec causing SIGSEGV)

**Content**: The note is about exploring retdec (a decompiler tool) after failing with mcsema/reopt tools. The user tested retdec on test programs and compared the output with anvill decompiler. They found the two LLVM IR files were not comparable - anvill's output was longer and less readable.

**Tags to extract**: retdec, 反编译 (decompilation), SIGSEGV, LLVM IR, 工具对比 (tool comparison), anvill

**Summary**: This note documents an exploration of retdec decompiler after other tools (mcsema/reopt) failed. The user compared retdec's LLVM IR output with anvill's output and found them incomparable.

**Key concepts**: 
- retdec: A decompiler tool
- LLVM IR: Intermediate representation used by both tools
- anvill: Another decompiler tool
- SIGSEGV: Segmentation fault error
- bin2llvmir: retdec's binary to LLVM IR conversion step

**Related notes**: Looking at the list, I need to find notes that might be related to reverse engineering, decompilation, or low-level system exploration. Most notes seem to be about MySQL, Clickhouse, database topics. None seem directly related to decompilation or reverse engineering tools.

Let me format this properly in Chinese.## 标签
retdec, 反编译, SIGSEGV, LLVM IR, anvill, 工具对比

## 摘要
记录在 mcsema/reopt 工具失败后重新探索 retdec 反编译器的过程。对比 retdec 和 anvill 生成的 LLVM IR 文件，发现两者输出差异大，anvill 生成的文件更长且可读性更差。

## 关键概念
- retdec: 开源可执行文件反编译器，可将二进制文件转换为 LLVM IR
- anvill: 基于 IDA Pro 的反编译工具，生成 JSON spec 再转换为 LLVM IR
- LLVM IR: 两种工具共同输出的中间表示格式，用于后续分析
- bin2llvmir: retdec 反编译流程中的中间步骤，将二进制转换为 LLVM

## 关联笔记
无
