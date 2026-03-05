---
note: 01KJBYN1G1KWW48KRK9Z9RR1NB.md
title: 20220915 - 电信IO分析 blktrace
indexed_at: 2026-03-05T08:08:51.890942+00:00
---

## 摘要
记录使用 blktrace 工具分析电信服务器 IO 延迟异常的过程。通过 btt 人类可读输出发现 IO 请求在设备上卡顿后集中完成的现象，定位 D2C 延迟高的问题。

## 关键概念
- blktrace: Linux 块设备 IO 追踪工具，用于捕获和分析块层 IO 事件
- btt: blktrace 后处理工具，将 IO 事件转换为人类可读的延迟分析
- D2C 延迟: 从 IO 请求下发 (Driver) 到完成 (Completion) 的总耗时
- IO 并发: 多个不同时间发出的 IO 请求在同一时刻完成的现象

## 关联笔记
- 01KJBYD9V2AQHGQD00WZFNHA7A.md: 同属 Linux IO 性能分析工具，讲解 iostat 与 sar 的数据来源和区别
- 01KJBZ7SQMZGYSPHGCPSH9BVB6.md: 涉及数据库 IO 瓶颈诊断场景，提到使用 iostat 观测 iowait 达 10-20% 的 IO 瓶颈问题
