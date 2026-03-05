---
note: 01KJBYVPR3SFP3J23BJMBGW4NV.md
title: 20221121 - 麒麟V10编译MySQL 8.0报错
indexed_at: 2026-03-05T08:16:04.629581+00:00
---

## 摘要
记录在麒麟 V10 系统上编译 MySQL 8.0 时遇到的 GCC 版本兼容性问题。因 devtoolset 安装失败（包依赖和签名问题），最终通过 Docker 使用 CentOS 8 环境成功编译。

## 关键概念
- devtoolset: Red Hat 提供的开发者工具集，包含新版 GCC 等编译工具
- 包签名冲突: 麒麟系统包与官方源包签名不同导致无法混用
- vault.centos.org: CentOS 归档仓库，用于访问已停止维护的 CentOS 版本源

## 关联笔记
- 01KJBYXQ5GXWFSE6D0CVP2MDXM.md: 记录 MySQL 编译配置信息，包含 devtoolset-8 编译参数
- 01KJBYTEBN37E9CRBHC043PV1E.md: 麒麟 V10 系统上 MySQL 在 ARM/x86 架构性能差异分析
- 01KJBYXQNR1PE2DGTP93ETQCDJ.md: MySQL 8.0.30 崩溃分析，同样使用 devtoolset-8 编译环境
