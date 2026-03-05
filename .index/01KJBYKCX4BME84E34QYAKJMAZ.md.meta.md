---
note: 01KJBYKCX4BME84E34QYAKJMAZ.md
title: 20220707 - 观察MySQL对 cgroup IO限制 造成压力的来源
indexed_at: 2026-03-05T08:04:56.064222+00:00
---

## 摘要
介绍如何使用 biosnoop 和 debugfs 工具分析 MySQL 在 cgroup IO 限制下的文件读写压力来源。通过扇区 -inode- 文件名的双向映射方法定位具体 IO 操作对应的文件，并分析 kworker/jbd2 进程在 cgroup 限制中的计入情况。

## 关键概念
- biosnoop: BCC 工具，用于监控块设备 IO 操作，显示进程、扇区、字节数和延迟
- cgroup blkio: Linux 控制组块 IO 限制机制，可限制 IOPS 和带宽
- inode: 文件系统中文件的唯一标识符，用于关联文件名与磁盘块
- debugfs: ext 文件系统调试工具，支持 icheck/ncheck 查询块号与 inode 映射
- extent: 文件在磁盘上的连续块分配单元，描述文件物理存储位置

## 关联笔记
- 01KJBZ3FQT8FKX47BF6Q8C4X7J.md: 同样涉及进程文件描述符分析和 IO 排查方法
- 01KJC00P6GGFBQ7N84E68RP3C8.md: 涉及 cgroups 配置相关内容（nvidia-container-cli.no-cgroups）
