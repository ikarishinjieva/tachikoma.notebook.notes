---
note: 01KJBYEAES2493TGQA8FEAXHPY.md
title: 20210814 - Clickhouse 编译
indexed_at: 2026-02-27T10:08:42.665362+00:00
---

The user wants me to analyze a note about ClickHouse compilation and generate index metadata in a specific Markdown format. Let me analyze the content:

1. The note is about compiling ClickHouse from source
2. It includes installing LLVM 11 and building ClickHouse with specific cmake settings
3. Looking at related notes, I can see several ClickHouse-related notes that would be relevant

Let me create the metadata:

Tags: ClickHouse, 编译, LLVM, CMake, 源码构建

Summary: 记录 ClickHouse v21.3.10.1-LTS 版本的源码编译过程。包括安装 LLVM 11 编译器和使用 CMake/Ninja 构建系统的完整步骤。

Key concepts:
- LLVM 11: ClickHouse 编译所需的编译器版本
- CMake: 用于配置 ClickHouse 构建选项的跨平台构建系统
- Ninja: 用于实际编译的构建工具
- Debug 构建: 使用 CMAKE_BUILD_TYPE=Debug 进行调试版本编译

Related notes:
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 都是 ClickHouse 相关学习笔记
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 都是 ClickHouse 相关学习笔记
- 01KJBYDBWTJGYAJWBZ93943E5R.md: 都是 ClickHouse 相关学习笔记
- 01KJBYDD58AJTW5NKT3GET7B89.md: 都是 ClickHouse 相关学习笔记
- 01KJBYDDA1R8E2ZZK7ERJ8BKKM.md: 都是 ClickHouse 相关学习笔记


- 01KJBYDBWTJGYAJWBZ93943E5R.md: ClickHouse Join 研究，同属 ClickHouse 技术栈
- 01KJBYDD58AJTW5NKT3GET7B89.md: ClickHouse+学习笔记，存在直接关联

I should narrow this down to the 2-3 most relevant connections rather than listing all of them.## 标签
ClickHouse, 编译, LLVM, CMake, 源码构建, Debug

## 摘要
记录 ClickHouse v21.3.10.1-LTS 版本的源码编译过程。包括安装 LLVM 11 编译器和使用 CMake/Ninja 构建系统的完整步骤，配置为 Debug 模式便于调试。

## 关键概念
- LLVM 11: ClickHouse 编译所需的指定版本编译器
- CMake: 跨平台构建系统，用于配置编译选项和编译器路径
- Ninja: 轻量级构建工具，用于执行实际编译任务
- Debug 构建: 通过 CMAKE_BUILD_TYPE=Debug 启用调试符号便于开发调试

## 关联笔记
- 01KJBYDAQ6YMA4RN9VCSAXHRZ0.md: 同为 ClickHouse 集群相关学习笔记
- 01KJBYDAYBCJZXBGJFXXQ48F5X.md: 同为 ClickHouse 引擎相关学习笔记
- 01KJBYDD58AJTW5NKT3GET7B89.md: ClickHouse 综合学习笔记，内容直接相关
