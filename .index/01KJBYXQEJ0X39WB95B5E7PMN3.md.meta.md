---
note: 01KJBYXQEJ0X39WB95B5E7PMN3.md
title: 20220826 - 百胜slave延迟, sar探索
indexed_at: 2026-02-27T10:22:46.154681+00:00
---

用户希望我分析这篇关于百胜 slave 延迟和 sar 探索的笔记，生成索引元数据。让我仔细阅读笔记内容并提取关键信息。

笔记主要内容：
1. 关于百胜 slave 延迟问题的分析
2. 使用 sar 工具进行系统性能分析
3. 涉及延迟、上下文切换、pgpgout/s、CPU 异常等多个维度的监控数据
4. 包含多个日期的监控图表（20 日、21 日、26 日、27 日等）
5. 涉及 Linux 性能指标分析（pgpgout、fault、bwrtn/s、wtps、tps 等）
6. 参考了 RedHat 文档和 Linux 源码

关键概念：
- sar: 系统活动报告工具
- pgpgout: 页面换出统计
- 上下文切换
- IO 指标（await、svctm、%util 等）

关联笔记分析：
- 01KJBYD9V2AQHGQD00WZFNHA7A.md: 20210409 - iostat 与 sar -b 的区别 - 都与 sar 和 IO 监控相关
- 01KJBYDAF12RN18BECZG1QSTEC.md: 20210519 - 内存去哪儿了 - 涉及内存分析，与 pgpgout 相关
- 01KJBYDB30D5DK9MDJ70R5YFST.md: 20210513 - 百胜 cpu 高的探索 - 同一项目（百胜）的性能问题探索


- 01KJBYDCFBDMBDD2VCZF2DZQX8.md: 20210606 - 从 binlog 中分离 SQL, 进行统计 - 可能涉及 slave 相关

现在生成索引。## 标签
MySQL, 性能分析, sar, slave 延迟, IO 监控, 上下文切换

## 摘要
记录百胜 MySQL slave 延迟问题的排查过程，通过 sar 工具分析多日监控数据。发现 pgpgout 峰值与延迟曲线上升起点重合，CPU 切换时间与延迟起止时间重合，涉及内存换页、IO 请求、上下文切换等多维度指标分析。

## 关键概念
- pgpgout: 每秒从内存换出的页面数，与延迟峰值趋势一致
- await/svctm: IO 延迟指标，写入量增多时反而变低
- 上下文切换: 偶数/奇数 CPU 切换时间与延迟发生时间重合
- sar: Linux 系统活动报告工具，用于采集性能监控数据

## 关联笔记
- 01KJBYD9V2AQHGQD00WZFNHA7A.md: iostat 与 sar -b 的区别，同属 IO 监控工具分析
- 01KJBYDB30D5DK9MDJ70R5YFST.md: 百胜 cpu 高的探索，同一项目的性能问题排查
- 01KJBYDAF12RN18BECZG1QSTEC.md: 内存去哪儿了，涉及内存分析与 pgpgout 相关
