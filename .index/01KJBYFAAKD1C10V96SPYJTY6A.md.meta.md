---
note: 01KJBYFAAKD1C10V96SPYJTY6A.md
title: 20211116 - mcsema 编译 和 使用
indexed_at: 2026-02-27T10:13:32.080291+00:00
---

用户要求我分析这篇关于 mcsema 编译和使用的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇技术笔记，记录了 mcsema 工具的编译和使用过程
2. 涉及到的主要技术：mcsema、remill、anvill、LLVM、IDA Pro
3. 记录了编译过程中的问题和解决方案
4. 记录了使用过程中的报错和尝试

需要生成的内容：
- 标签：3-6 个关键词
- 摘要：2-3 句话
- 关键概念：3-5 个核心概念
- 关联笔记：从提供的笔记列表中找关联的（看起来没有直接关联的）

看其他笔记列表，主要是关于 MySQL、Clickhouse、性能分析等主题，与这篇 mcsema 二进制提升工具的笔记没有直接关联。## 标签
mcsema, 二进制提升, LLVM, IDA Pro, 编译问题, 逆向工程

## 摘要
记录了 mcsema 工具链（remill/anvill/mcsema）的完整编译流程及遇到的 LLVM 版本兼容问题。详细描述了使用 mcsema-disass 和 mcsema-lift 对 MySQL 二进制文件进行提升时的多个报错及排查过程，最终因 anvill 层错误无法继续。

## 关键概念
- mcsema: 将 x86 二进制代码提升到 LLVM IR 的工具
- remill: 二进制代码翻译器，将机器码转换为 LLVM IR
- anvill: 提供程序分析和优化能力的库
- PIE-mode: 位置无关可执行模式，处理地址随机化的二进制文件
- CFG: 控制流图，描述程序执行路径的结构

## 关联笔记
无
