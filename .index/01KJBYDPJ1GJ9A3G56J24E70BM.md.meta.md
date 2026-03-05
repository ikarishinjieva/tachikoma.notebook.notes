---
note: 01KJBYDPJ1GJ9A3G56J24E70BM.md
title: 20210716 - CPU max MHz 的解释
indexed_at: 2026-03-05T07:35:22.690478+00:00
---

## 标签
CPU 频率，cpufreq, lscpu，硬件性能，Linux 系统

## 摘要
笔记分析了 Linux 系统中 lscpu 命令显示的 Max CPU MHz 数据来源，确认其来自 cpufreq 而非 cpuinfo。通过对比 cpuinfo 与 cpufreq 的实际值，结合 Intel 官网规格，解释了 CPU 当前频率、最高频率与型号标称频率之间的差异。

## 关键概念
- cpufreq: Linux 内核提供的 CPU 频率调节子系统，提供动态频率调整
- lscpu: util-linux 工具包中的 CPU 信息查看命令，从 cpufreq 读取最大频率
- cpuinfo: /proc/cpuinfo 文件，提供静态 CPU 信息但不包含动态频率数据
- 基准频率: CPU 型号标称的工作频率（如 2.2 GHz），区别于睿频上限

## 关联笔记
无
