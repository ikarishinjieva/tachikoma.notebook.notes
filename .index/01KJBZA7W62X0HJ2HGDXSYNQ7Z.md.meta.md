---
note: 01KJBZA7W62X0HJ2HGDXSYNQ7Z.md
title: 20240620 - ChatDBA: 如何判断参考材料和问题的相关性 [4]
indexed_at: 2026-03-05T10:13:49.231808+00:00
---

## 摘要
测试 ChatDBA 新流程对 MySQL 主从延迟问题的处理效果，通过三个参考文档验证相关性判断能力。结果显示召回文档均与主从延迟高度相关，流程能正确识别大事务、单线程复制、时间同步等关键因素并生成排查思路。

## 关键概念
- Seconds_Behind_Master: MySQL 从库延迟的核心监控指标
- 并行复制: 通过多线程复制提升从库应用 binlog 的速度
- 大事务: 长时间未提交或批量操作事务，易导致从库复制延迟
- 思维链: LLM 逐步推理生成排查思路的方法
- 单线程复制: slave_parallel_workers=0 的复制方式，处理大事务时易产生延迟

## 关联笔记
- 01KJBZA51RVTP0C0VXYHQ96C2D.md: 同一系列的相关性判断测试笔记 [1]
- 01KJBZA6T5H1DQDV560RRNDGK7.md: 同一系列的相关性判断测试笔记 [3]
- 01KJBZD8WG7RH0DJMMACNG562Q.md: 涉及主从复制错误的排查案例
