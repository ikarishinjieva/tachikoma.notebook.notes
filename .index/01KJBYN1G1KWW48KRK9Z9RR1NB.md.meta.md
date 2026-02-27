---
note: 01KJBYN1G1KWW48KRK9Z9RR1NB.md
title: 20220915 - 电信IO分析 blktrace
indexed_at: 2026-02-27T10:18:32.029035+00:00
---

用户希望我分析这篇关于电信 IO 分析的笔记，生成索引元数据。让我仔细阅读笔记内容：

1. 笔记标题：20220915 - 电信 IO 分析 blktrace
2. 内容涉及：
   - blktrace 工具使用
   - IO 延迟分析（D2C 延迟）
   - btt 输出分析
   - IO 设备卡顿问题排查
   - 参考了 Red Hat 性能调优文档

3. 关联笔记分析：
   - 01KJBYD9V2AQHGQD00WZFNHA7A.md: MySQL 刷盘研究 - 数据文件刷盘慢导致 SQL 卡顿（与 IO 延迟相关）
   - 01KJBYD9V2AQHGQD00WZFNHA7A.md 和 01KJBYD9S0A3VZWG60QKDTAF44.md 都涉及 IO/刷脏页相关
   - 01KJBYD9V2AQHGQD00WZFNHA7A.md: iostat 与 sar -b 的区别（IO 监控工具）

让我生成合适的索引：

标签：blktrace, IO 分析，性能调优，延迟排查，存储

摘要：记录使用 blktrace 分析电信 IO 延迟问题的过程。通过 btt 输出发现 IO 设备存在卡顿现象，多个 IO 请求在同一时间完成。

关键概念需要涵盖 blktrace 的核心机制：D2C 延迟衡量 IO 交互时间，btt 提供可读的 trace 输出，而 blktrace 本身用于追踪块设备 IO 请求的完整生命周期。

关联笔记方面，01KJBYD9VE4NJQXE9P0VKM16Q4.md 涉及 MySQL 刷盘与数据文件慢的问题，01KJBYD9V2AQHGQD00WZFNHA7A.md 则对比了 iostat 与 sar -b 两种 IO 监控工具，这些都与 IO 性能分析相关。## 标签
blktrace, IO 分析，性能调优，延迟排查，存储

## 摘要
记录使用 blktrace 分析电信 IO 延迟问题的过程。通过 btt 输出发现 IO 设备存在卡顿现象，多个不同时间发出的 IO 请求在同一时间完成。

## 关键概念
- D2C: 从提交到完成的 IO 延迟时间
- btt: blktrace 的人类可读输出工具
- blktrace: Linux 块设备 IO 追踪工具

## 关联笔记
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 都涉及存储刷盘慢导致卡顿的 IO 问题研究
- 01KJBYD9V2AQHGQD00WZFNHA7A.md: 同为 IO 监控工具对比分析
