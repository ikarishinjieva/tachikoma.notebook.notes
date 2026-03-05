---
note: 01KJBYEAES2493TGQA8FEAXHPY.md
title: 20210814 - Clickhouse 编译
indexed_at: 2026-03-05T07:40:06.097917+00:00
---

## 摘要
记录 ClickHouse v21.3.10.1-lts 版本的源码编译流程，包括 LLVM 11 安装、git 子模块初始化、cmake 配置和 ninja 构建。使用 socks5 代理克隆仓库，指定 clang-11 作为编译器，采用 Debug 模式构建。

## 关键概念
- LLVM 11: 编译器基础设施框架，ClickHouse 编译需要 clang-11 作为 C/C++ 编译器
- cmake: 跨平台构建系统生成器，用于配置 ClickHouse 编译选项
- ninja: 小型构建系统，专注于速度，用于实际编译过程
- Debug 模式: CMAKE_BUILD_TYPE 设置为 Debug，生成带调试信息的二进制文件

## 关联笔记
- 01KJBYXQWQNYSHP23Q9B3MDJCR.md: 同属 ClickHouse 主题，记录 ClickHouse Too many parts 报错和 AST is too big 错误排查
- 01KJBYF973H98W1RNZ05SS50KE.md: 涉及 LLVM IR 和 clang 编译 MySQL，与当前笔记的 LLVM/clang 编译工具链相关
