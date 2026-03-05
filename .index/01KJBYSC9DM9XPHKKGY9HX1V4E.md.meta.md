---
note: 01KJBYSC9DM9XPHKKGY9HX1V4E.md
title: 20221102 - 更新内核后, 服务器网卡驱动丢失
indexed_at: 2026-03-05T08:13:16.781351+00:00
---

## 标签
Linux 内核，网卡驱动，驱动编译，GCC，Broadcom，故障排查

## 摘要
记录内核 4.x 升级到 5.x 后 Broadcom NXE 网卡驱动丢失的修复过程。通过更换 GCC 版本解决 define 语法错误，修改驱动代码适配 ib_umem_get 函数签名变更完成编译。

## 关键概念
- Linux 内核 API 兼容性: 4.x 到 5.x 版本间内核函数定义发生变更
- GCC 编译器版本: 低版本不支持 define 宏中带空格的语法
- ib_umem_get: RDMA 相关内核函数，5.x 内核中函数签名有变更
- insmod: Linux 内核模块手动加载命令

## 关联笔记
- 01KJBYN0PMGKH7VEMV0SZSE9PR.md: 涉及 Linux 内核版本相关的仓库资源
