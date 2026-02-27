---
title: 20220819 - 如何理解perf event: node-load-misses
confluence_page_id: 1933710
created_at: 2022-08-18T16:42:20+00:00
updated_at: 2022-08-18T16:42:20+00:00
---

查看Linux源码: <https://github.com/torvalds/linux/blob/master/arch/x86/events/intel/core.c>

找到Skylake的hardware cache event定义: skl_hw_cache_event_ids

![image2022-8-19 0:38:55.png](/assets/01KJBYM9JN59CM9N2ZHKRYFQ2R/image2022-8-19%200%3A38%3A55.png)

配合skl_hw_cache_extra_regs: 

![image2022-8-19 0:39:25.png](/assets/01KJBYM9JN59CM9N2ZHKRYFQ2R/image2022-8-19%200%3A39%3A25.png)

node-load 的标志是 L3_MISS_LOCAL_DRAM

node-load-misses 的标志是 L3_MISS_REMOTE

看上去一个是本地读, 一个是远程读
