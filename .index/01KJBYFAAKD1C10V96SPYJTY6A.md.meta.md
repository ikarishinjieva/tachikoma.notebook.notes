---
note: 01KJBYFAAKD1C10V96SPYJTY6A.md
title: 20211116 - mcsema 编译 和 使用
indexed_at: 2026-03-05T07:49:46.161350+00:00
---

## 摘要
记录 mcsema 工具链（remill、anvill、mcsema）在 Ubuntu 18.04 上的编译过程及遇到的 LLVM 版本兼容性问题。使用 IDA Pro 配合 mcsema-disass 进行二进制文件反编译时遇到 protobuf 版本冲突和 anvill lift 错误，最终未能成功完成二进制到 LLVM IR 的转换。

## 关键概念
- mcsema: 将 x86/AMD64 二进制代码提升为 LLVM IR 的工具
- anvill: 提供 ABI 规范和内存模型，配合 mcsema 使用
- remill: 二进制代码提升的基础库，提供指令语义
- LLVM IR: LLVM 的中间表示格式，用于代码优化和分析
- IDA Pro: 商业反汇编工具，用于生成 mcsema 所需的 CFG 文件

## 关联笔记
- 01KJBYF973H98W1RNZ05SS50KE.md: 记录 anvill 反编译报错及 retdec 生成的 LLVM IR 问题
- 01KJBYFBDSR1SNE0X711Z0TGXR.md: 尝试 mcsema/reopt 失败后转向 retdec 的探索过程
- 01KJBYEAES2493TGQA8FEAXHPY.md: LLVM 11 安装配置的相关记录
