---
note: 01KJBYKCX4BME84E34QYAKJMAZ.md
title: 20220707 - 观察MySQL对 cgroup IO限制 造成压力的来源
indexed_at: 2026-02-27T10:17:40.421600+00:00
---

用户希望我分析这篇关于 MySQL 和 cgroup IO 限制的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记主题：使用 cgroup 限制 MySQL IO 时，如何分析是哪些文件的读写造成了 IO 压力
2. 主要内容：
   - 使用 biosnoop 工具分析 IO
   - 如何通过扇区号查找对应文件
   - 如何通过文件查找对应扇区
   - 使用 stackcount 分析写入堆栈

3. 关键概念：
   - biosnoop: IO 监控工具
   - cgroup IO 限制
   - 扇区/块号/inode 映射
   - debugfs 工具
   - stackcount 堆栈分析

4. 关联笔记分析：
   - 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: 20210404 - MySQL 刷脏页研究 - 与 IO/刷页相关
   - 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 20210329 - MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 - 与 IO/刷盘相关
   - 01KJBYD9V2AQHGQD00WZFNHA7A.md: 20210409 - iostat 与 sar -b 的区别 - 与 IO 监控相关

让我生成索引：

标签：MySQL, cgroup, IO 监控，biosnoop, 文件系统，debugfs


摘要：这篇笔记阐述了在 cgroup 限制 MySQL IO 的场景下，如何通过 biosnoop 定位造成压力的具体文件。介绍了扇区号与文件的双向映射方法，并用 stackcount 追踪写入操作的调用栈。

关键概念：
- biosnoop: bcc 工具集的 IO 监控组件，追踪磁盘读写操作
- 扇区映射：通过 fdisk、tune2fs、debugfs 等工具实现扇区号与文件路径的转换
- debugfs: ext 文件系统的调试工具，支持 inode 与文件名的互查
- stackcount: bcc 工具，用于统计内核函数的调用堆栈

关联笔记：
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 涉及 MySQL IO 刷页机制
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 - 涉及 IO 压力分析
- 01KJBYD9V2AQHGQD00WZFNHA7A.md: iostat 与 sar -b 的区别 - 涉及 IO 监控工具## 标签
MySQL, cgroup, IO 监控，biosnoop, debugfs, 扇区映射

## 摘要
本文介绍在使用 cgroup 限制 MySQL IO 时，如何分析哪些文件的读写造成了 IO 压力。通过 biosnoop 获取扇区信息，利用 fdisk/tune2fs/debugfs 工具链实现扇区与文件的双向映射，并结合 stackcount 分析写入堆栈定位 IO 来源。

## 关键概念
- biosnoop: bcc 工具，用于监控磁盘 IO 请求，输出进程/扇区/字节/延迟等信息
- 扇区映射: 通过 fdisk 获取分区偏移、tune2fs 获取块大小、debugfs 查找 inode 实现扇区到文件的定位
- debugfs: ext 文件系统调试工具，通过 icheck/ncheck/stat 命令查询块号与 inode/文件名的对应关系
- stackcount: bcc 工具，统计内核函数（如 submit_bio）的调用堆栈，分析 IO 来源

## 关联笔记
- 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 涉及 MySQL 后台刷页与 IO 压力分析
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿 - 同属 MySQL IO 问题排查
- 01KJBYD9V2AQHGQD00WZFNHA7A.md: iostat 与 sar -b 的区别 - 涉及 IO 监控工具对比
