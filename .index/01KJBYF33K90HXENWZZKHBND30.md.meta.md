---
note: 01KJBYF33K90HXENWZZKHBND30.md
title: 20211028 - retdec 试用
indexed_at: 2026-03-05T07:47:24.804690+00:00
---

## 摘要
记录使用 retdec 工具将 binary code 转换为 LLVM IR 并进行优化的试用过程。包含各 phase 执行时间日志、bc 文件格式解析及生成的 C 代码示例。

## 关键概念
- retdec: 反编译工具，将 binary code 转为 LLVM IR 再优化回 binary code
- LLVM IR: 中间表示格式，retdec 用于存储和处理反编译结果
- bin2llvmir: LLVM passes 库，用于将 binaries 翻译为 LLVM IR modules
- bc 文件: LLVM IR 的二进制存储格式
- 优化 phases: retdec 执行的多阶段优化流程（如栈优化、常量优化等）

## 关联笔记
- 01KJBYF973H98W1RNZ05SS50KE.md: 记录 retdec 生成的 LLVM IR 文件编译时报 SEGSIGV 错误的问题
- 01KJBYFBDSR1SNE0X711Z0TGXR.md: 使用 retdec 和 anvill 对测试程序进行反编译的实践
