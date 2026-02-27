---
note: 01KJBYF33K90HXENWZZKHBND30.md
title: 20211028 - retdec 试用
indexed_at: 2026-02-27T10:12:28.971029+00:00
---

用户要求我分析这篇关于 retdec 试用的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：retdec 工具试用，用于 binary code 到 LLVM IR 的反编译和优化
2. 核心内容：
   - retdec 的目的：binary code -> LLVM IR -> 优化的 binary code
   - 展示了 retdec 各个 phase 的执行日志
   - 解释了 .bc 文件是 LLVM IR 的存储形式
   - 展示了如何用 llvm-bcanalyzer 分析 bc 文件
   - 提到了生成的 mysqlbinlog.c 文件

3. 标签应该包括：retdec, 反编译, LLVM IR, binary optimization, mysqlbinlog 等

4. 关联笔记：看其他笔记列表，主要是关于 MySQL、Clickhouse、数据库相关的，这篇笔记是关于 binary 反编译的，看起来没有直接关联的笔记。

让我生成索引：

## 标签
retdec, 反编译, LLVM IR, binary optimization, mysqlbinlog, bin2llvmir

## 摘要
记录使用 retdec 工具将 mysqlbinlog 二进制文件反编译为 LLVM IR 的过程。分析了 retdec 各优化阶段及生成的.bc 文件格式，展示了 llvm-bcanalyzer 的解析方法。

## 关键概念
- retdec: 二进制代码反编译器，可将 binary code 转为 LLVM IR


- bin2llvmir: 将二进制文件翻译为 LLVM IR modules 的 LLVM passes 库
- .bc 文件: LLVM IR 的位码存储格式
- llvm-bcanalyzer: 用于解析和分析.bc 文件的工具

## 关联笔记
无（其他笔记主要是数据库相关，与 binary 反编译无直接关联）
## 标签
retdec, 反编译, LLVM IR, binary optimization, mysqlbinlog, bin2llvmir

## 摘要
记录使用 retdec 工具将 mysqlbinlog 二进制文件反编译为 LLVM IR 的试用过程。分析了 retdec 各优化阶段的执行日志，以及.bc 文件格式和 llvm-bcanalyzer 解析方法。

## 关键概念
- retdec: 二进制代码反编译器，可将 binary code 转为 LLVM IR 再进行优化
- bin2llvmir: 将二进制文件翻译为 LLVM IR modules 的 LLVM passes 库
- .bc 文件: LLVM IR 的位码（Bitcode）存储格式
- llvm-bcanalyzer: LLVM 提供的.bc 文件分析工具

## 关联笔记
无
