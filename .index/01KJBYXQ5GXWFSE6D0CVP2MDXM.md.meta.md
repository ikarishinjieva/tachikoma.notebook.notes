---
note: 01KJBYXQ5GXWFSE6D0CVP2MDXM.md
title: 20230110 - MySQL 编译信息
indexed_at: 2026-03-05T08:19:46.993445+00:00
---

## 摘要
记录 MySQL 8.0.30 的完整编译配置信息，包括构建环境（Linux、cmake 3.11.2）和详细的 CMake 特性标志。可用于复现相同编译环境或分析编译参数对性能的影响。

## 关键概念
- CMAKE_BUILD_TYPE: 编译类型配置，RelWithDebInfo 表示带调试信息的发布版本
- Feature Flags: CMake 配置选项，控制 MySQL 功能模块的启用/禁用状态
- devtoolset-8: Red Hat 开发者工具集，提供更新版本的 GCC 编译器链
- INSTALL_LAYOUT: 安装布局配置，STANDALONE 表示独立安装模式

## 关联笔记
- 01KJBYHCENW7J7KPJMGXJGAK4S.md: 工行 MySQL crash 分析，提到 MySQL 实例是独自编译的，可对比编译配置
- 01KJBZ7XD3F9GQ1H1VCB71FRMY.md: 对比 Mariadb client 和 obclient 的差异，涉及 INFO_BIN 和 CMake 配置
- 01KJBYF973H98W1RNZ05SS50KE.md: MySQL LLVM IR 编译实验，涉及底层编译优化研究
