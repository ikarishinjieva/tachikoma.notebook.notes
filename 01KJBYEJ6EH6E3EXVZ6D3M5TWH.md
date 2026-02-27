---
title: 20210828 - THP 文档
confluence_page_id: 1343686
created_at: 2021-08-28T04:13:03+00:00
updated_at: 2021-09-05T10:06:04+00:00
---

- <https://access.redhat.com/solutions/46111>
    - 检查进程级别THP的使用量

```
    # awk  '/AnonHugePages/ { if($2>4){print FILENAME " " $0; system("ps -fp " gensub(/.*\/([0-9]+).*/, "\\1", "g", FILENAME))}}' /proc/*/smaps
/proc/7519/smaps:AnonHugePages:    305152 kB
UID        PID  PPID  C STIME TTY          TIME CMD
qemu      7519     1  1 08:53 ?        00:00:48 /usr/bin/qemu-system-x86_64 -machine accel=kvm -name rhel7 -S -machine pc-i440fx-1.6,accel=kvm,usb=of
/proc/7610/smaps:AnonHugePages:    491520 kB
UID        PID  PPID  C STIME TTY          TIME CMD
qemu      7610     1  2 08:53 ?        00:01:30 /usr/bin/qemu-system-x86_64 -machine accel=kvm -name util6vm -S -machine pc-i440fx-1.6,accel=kvm,usb=
/proc/7788/smaps:AnonHugePages:    389120 kB
UID        PID  PPID  C STIME TTY          TIME CMD
qemu      7788     1  1 08:54 ?        00:00:55 /usr/bin/qemu-system-x86_64 -machine accel=kvm -name rhel64eus -S -machine pc-i440fx-1.6,accel=kvm,us
```

    - 禁用THP, 对已经分配出去的内存无效
      - Running the above commands will stop only creation and usage of the new THP. The THP which were created and used at the moment the above commands were run would not be disassembled into the regular memory pages. To get rid of THP completely the system should be rebooted with THP disabled at boot time.
  - <https://en.pingcap.com/blog/why-we-disable-linux-thp-feature-for-databases>
    - THP的目的是 调大页大小, 改善TLB的命中率 (translation lookaside buffer)
  - <https://www.percona.com/blog/2019/03/06/settling-the-myth-of-transparent-hugepages-for-databases/>  

    - TLB-load-misses: 
      - 在数据库的场景下, TLB的命中率 在THP禁用/启用的情况下差异不大 ??
  - <https://partners-intl.aliyun.com/help/doc-detail/161963.htm>
    - THP配置: /sys/kernel/mm/transparent_hugepage/enabled
      - always
      - never
      - madvise: 对标记MADV_HUGEPAGE的内存, 开启THP的功能
    - THP 碎片整理配置: /sys/kernel/mm/transparent_hugepage/defrag (无法分配内存时, 是否进行碎片整理)
      - always: 
        - 无法分配THP时, 阻塞内存分配, 立刻触发内存整理, 直到有足够空间
      - defer: 
        - 无法分配THP时, 回退到分配4k内存, 并唤醒kswapd进行内存回收
      - madvise:
        - 对标记MADV_HUGEPAGE的内存, 开启碎片整理, 行为相当于always
      - defer+madvise:
        - 对标记MADV_HUGEPAGE的内存, 开启碎片整理, 行为相当于defer
      - never
        - 无法分配内存时, 不进行内存整理
    - khugepaged 碎片整理配置: (日常的内存整理行为)
      - /sys/kernel/mm/transparent_hugepage/khugepaged/defrag 
        - 0: 禁用
        - 1: khugepaged进程会将4KB页合并成2MB页
      - /sys/kernel/mm/transparent_hugepage/khugepaged/alloc_sleep_millisecs

        - THP分配失败后, 等待一段时间后, 才能再分配THP
      - /sys/kernel/mm/transparent_hugepage/khugepaged/scan_sleep_millisecs

        - khugepaged被唤醒的周期
      - /sys/kernel/mm/transparent_hugepage/khugepaged/pages_to_scan

        - khugepaged每次工作需要扫描的页数
    - THP的缺陷
      - 如果 transparent_hugepage/defrag 配置为 always, THP不足时, 申请内存的操作会阻塞, 导致性能下降
      - 如果开启 khugepaged, 碎片整理时会锁内存页, 影响性能
      - 当两类碎片整理都关掉时, THP可能比4k页消耗更快, 导致内存快速不足
    - THP配置建议
      - 建议 THP策略默认配置为 defer+madvise
      - 如果 khugepaged 导致 100% CPU占用, 可调整 scan_sleep_millisecs, 或关掉相关碎片整理
      - 对以下几类应用, 建议关闭THP
        - "database applications have a large number of access requests"
        - "a large number of latency-sensitive application scenarios"
        - "a large number of short-lived memory Allocation (Short-lived Allocation) scenarios"
  - <https://en.wikipedia.org/wiki/Translation_lookaside_buffer>
    - TLB的解释
  - <https://alexandrnikitin.github.io/blog/transparent-hugepages-measuring-the-performance-impact/>
    - TODO
  - [附件: Huge_pages_and_databases___FOSDEM19.pdf] 
  - <http://cenalulu.github.io/linux/huge-page-on-numa/>
    - TLB示意图:
      - ![image](http://cenalulu.github.io/images/linux/huge_page/tlb_lookup.png)
    - 通过`oprofile`抓取到`TLB Miss`导致的运行时间占程序总运行时间的多少，来计算出Huge Page所能带来的预期性能提升 ??
    - 在NUMA环境下, HugePage可能导致性能下降
      - CPU Cache写冲突 的概率增加  

      - 连续数据需要跨NUMA区访问
        - ![image](http://cenalulu.github.io/images/linux/huge_page/false_sharing.png)  

  - <https://cloud.tencent.com/developer/article/1485099>
  - <https://lwn.net/Articles/379748/> : A deeper look at TLBs and costs
    - TLB miss 带来的时间成本 = TLB miss的估算次数 * TLB miss 的估算成本
      - 估算次数
        - oprofile 可以评估进程级别的PMC统计
      - 估算成本
        - calibrator 用于测试机器的TLB miss成本
        - 可以使用PMC评估 (DTLB_MISS, DATA_TABLEWALK_CYC), 但需要一个稳定的压力, 可使用 STREAM 压力
        - 可以使用 tlbmiss_cost 评估TLB miss的成本
      - 直接评估: 
        - PMC: dtlb_store_misses.walk_duration
