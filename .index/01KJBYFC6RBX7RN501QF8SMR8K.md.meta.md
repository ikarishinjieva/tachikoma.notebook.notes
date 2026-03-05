---
note: 01KJBYFC6RBX7RN501QF8SMR8K.md
title: 20211118 - HDTrans 使用
indexed_at: 2026-03-05T07:50:48.199634+00:00
---

## 摘要
记录 HDTrans 动态二进制翻译工具在 32 位虚拟机上的编译配置与运行时 SIGSEGV 崩溃问题。通过 gdb 调试定位崩溃位置，分析 x86 机器码指令格式，尝试用 noop 替换崩溃指令段进行问题排查。

## 关键概念
- HDTrans: 动态二进制翻译工具，用于指令级翻译和动态优化
- SIGSEGV: 段错误信号，表示访问了无效内存地址
- Mod R/M: x86 指令编码格式中的操作数指定字节
- LEA 指令: Load Effective Address，用于计算内存地址
- UserEntry: HDTrans 中翻译器的入口函数

## 关联笔记
- 01KJBYGEDJ1QA5349NBVKGHQWG.md: 同一主题的后续笔记，继续记录 HDTrans 在 32 位虚拟机上的使用
- 01KJBYFBDSR1SNE0X711Z0TGXR.md: 同一天前后关于 SIGSEGV 调试的探索笔记
