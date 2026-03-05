---
note: 01KJBYGEDJ1QA5349NBVKGHQWG.md
title: 20220129 - HDTrans在32位虚拟机上的使用
indexed_at: 2026-03-05T07:54:30.012514+00:00
---

## 标签
HDTrans, 虚拟机, QEMU, VNC, SSH Tunnel, 二进制翻译

## 摘要
记录在 32 位 QEMU 虚拟机上使用 HDTrans 动态二进制翻译工具的完整环境配置和调试过程。遇到 isupper 函数 panic 问题，通过替换为 ASCII 判断解决；性能测试显示 HDTrans 仅提供低消耗翻译机制，无额外运行优化。

## 关键概念
- HDTrans: 动态二进制翻译工具，以 Basic Block 为单位翻译二进制代码
- QEMU: 开源虚拟机软件，用于搭建 32 位测试环境
- SSH Tunnel: 通过 SSH 加密隧道映射远程端口到本地
- VNC: 虚拟机远程桌面协议，用于图形化访问虚拟机
- Basic Block: 基本块，HDTrans 翻译二进制代码的基本单位

## 关联笔记
- 01KJBYFC6RBX7RN501QF8SMR8K.md: 同一系列 HDTrans 使用笔记，记录更早的 HDTrans 调试经验## 标签
HDTrans, 虚拟机, QEMU, VNC, SSH Tunnel, 二进制翻译

## 摘要
记录在 32 位 QEMU 虚拟机上使用 HDTrans 动态二进制翻译工具的完整环境配置和调试过程。遇到 isupper 函数 panic 问题，通过替换为 ASCII 判断解决；性能测试显示 HDTrans 仅提供低消耗翻译机制，无额外运行优化。

## 关键概念
- HDTrans: 动态二进制翻译工具，以 Basic Block 为单位翻译二进制代码
- QEMU: 开源虚拟机软件，用于搭建 32 位测试环境
- SSH Tunnel: 通过 SSH 加密隧道映射远程端口到本地
- VNC: 虚拟机远程桌面协议，用于图形化访问虚拟机
- Basic Block: 基本块，HDTrans 翻译二进制代码的基本单位

## 关联笔记
- 01KJBYFC6RBX7RN501QF8SMR8K.md: 同一系列 HDTrans 使用笔记，记录更早的 HDTrans 调试经验
