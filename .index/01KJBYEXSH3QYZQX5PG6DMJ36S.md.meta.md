---
note: 01KJBYEXSH3QYZQX5PG6DMJ36S.md
title: 20210923 - linux rc script 相关知识
indexed_at: 2026-02-27T10:10:48.221117+00:00
---

用户要求我分析这篇关于 Linux rc script 和 systemd 启动流程的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20210923 - linux rc script 相关知识
2. 内容涵盖：
   - Linux 5.x 启动流程（BIOS, Boot Loader, Kernel, /sbin/init, runlevel, /etc/inittab, rc.d/rc）
   - Systemd 启动流程（systemctl list-dependencies 输出，target 依赖关系）
   - systemd 与 runlevel 的对应关系
   - 如何改变 systemd 的 runlevel

3. 关联笔记分析：
   - 查看其他笔记列表，大部分是关于 MySQL、Clickhouse、数据库相关的
   - 这篇笔记是关于 Linux 系统启动流程的
   - 其他笔记中没有明显与 Linux 启动、systemd、runlevel 相关的内容
   - 所以关联笔记应该是「无」

现在生成元数据：

标签：Linux, systemd, 启动流程, runlevel, init, 系统管理

摘要：笔记记录了传统 Linux SysV init 的 runlevel 启动机制和现代 systemd 的启动流程。包含 systemctl 依赖树输出和 runlevel 切换方法。

关键概念：
- runlevel: 传统 SysV init 的系统运行级别，通过 K/S 脚本控制服务启停
- systemd target: systemd 中替代 runlevel 的目标单元，用于组织服务依赖


- systemctl: systemd 的服务管理命令，可设置默认启动目标
- SysV init: 传统的 Linux 初始化系统，通过/etc/inittab 和 rc.d 脚本管理启动

关联笔记：无（其他笔记主要是数据库相关，与 Linux 启动流程无直接关联）
## 标签
Linux, systemd, 启动流程, runlevel, init, 系统管理

## 摘要
笔记记录了传统 Linux SysV init 的 runlevel 启动机制和现代 systemd 的启动流程。包含 systemctl 依赖树输出示例和 runlevel 切换方法。

## 关键概念
- runlevel: 传统 SysV init 的系统运行级别，通过 K/S 脚本控制服务启停
- systemd target: systemd 中替代 runlevel 的目标单元，用于组织服务依赖关系
- systemctl: systemd 的服务管理命令，可设置默认启动目标
- SysV init: 传统 Linux 初始化系统，通过/etc/inittab 和 rc.d 脚本管理启动

## 关联笔记
无
