---
title: 20220910 - 百胜复制延迟, 再次整理线索 - 中断
confluence_page_id: 1933792
created_at: 2022-09-10T11:23:59+00:00
updated_at: 2022-10-04T16:42:15+00:00
---

# turbostat

延迟

![image2022-9-10 19:19:38.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-10%2019%3A19%3A38.png)

无延时，11:00

![image2022-9-10 19:21:46.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-10%2019%3A21%3A46.png)

11:18 15s延时

![image2022-9-10 19:22:5.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-10%2019%3A22%3A5.png)

去掉空闲连接

![image2022-9-10 19:20:32.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-10%2019%3A20%3A32.png)

观察到IRQ 在去掉空闲连接 和 无延时时, 都有 1/3 的提升

线索

  - /proc/interrupts
    - turbostat 解析的是 /proc/interrupts 的输出
    - 仅包括 硬件IRQ
  - /proc/softirqs
  - irqbalance

# 获取了interrupt信息

以下信息是 故障机器 在正常状态下的信息

[附件: 0914-proc-turbostat.tar.gz] 

整理信息后, 发现CAL和LOC高: 

![image2022-9-14 21:2:52.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-14%2021%3A2%3A52.png)

NUMA-0的IRQ高: 

![image2022-9-14 21:3:20.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-14%2021%3A3%3A20.png)

原始数据文件: 

[turbostat.xlsx](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/turbostat.xlsx)

找到相关线索: 

  - <https://www.codetd.com/en/article/12032796>
  - kernel.timer_migration bug: <https://bugzilla.kernel.org/show_bug.cgi?id=124661>

观察手段: dstat -t -i -I RES,CAL,TLB -y

参考: 

  - MSI 和 INT的差别: <https://blog.csdn.net/huangkangying/article/details/11178425>
    - Message Signaled Interrupt
    - Pin-based interrupt
    - 文章有两种方式的优劣对比
  - #### Processing of hardware interrupts in Linux

    - <http://events17.linuxfoundation.org/sites/events/files/slides/interrupts_16x9_final.pdf>
    - irqbalance的工作机制
  - CAL (Function call interrupt) 的解释 线索  

    - <https://access.redhat.com/solutions/5440761>
    - <https://github.com/torvalds/linux/blob/64a925c9271ec50714b9cea6a9980421ca65f835/arch/x86/kernel/smp.c>

```
DEFINE_IDTENTRY_SYSVEC(sysvec_call_function)
{
	ack_APIC_irq();
	trace_call_function_entry(CALL_FUNCTION_VECTOR);
	inc_irq_stat(irq_call_count);
	generic_smp_call_function_interrupt();
	trace_call_function_exit(CALL_FUNCTION_VECTOR);
}
```

      - inc_irq_stat(irq_call_count) 增加统计值
      - 诊断命令: 
        - trace-bpfcc smp_call_function_interrupt
        - stackcount-bpfcc smp_call_function_interrupt

# 诊断CAL的来源

trace-bpfcc smp_call_function_interrupt

![image2022-9-19 13:43:6.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-19%2013%3A43%3A6.png)

...

# Linux源码分析CAL的机制

操作系统版本

![image2022-9-30 10:49:30.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-30%2010%3A49%3A30.png)

找到状态累加的代码

![image2022-9-22 13:36:22.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-22%2013%3A36%3A22.png)

smp_call_function_interrupt 被注册为 APIC interrupt: 

![image2022-9-22 13:58:30.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-22%2013%3A58%3A30.png)

### 查找中断的生产方: 

查找 CALL_FUNCTION_VECTOR, 发现触发中断的生产方: 

![image2022-9-22 14:16:58.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-22%2014%3A16%3A58.png)

查找 native_send_call_func_ipi

![image2022-9-22 14:18:39.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-22%2014%3A18%3A39.png)

查找 send_call_func_ipi: 

![image2022-9-22 14:19:30.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-22%2014%3A19%3A30.png)

查找 arch_send_call_function_ipi_mask, 找到 smp_call_function_many

查找 smp_call_function_many:

<https://elixir.bootlin.com/linux/v3.10/C/ident/smp_call_function_many>

可以找到各触发点, 包括TLB等: 

![image2022-9-22 14:21:43.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-22%2014%3A21%3A43.png)

### 通过 call_single_queue 查找生产方

  - generic_exec_single
    - ![image2022-9-22 13:53:31.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-22%2013%3A53%3A31.png)
  - smp_call_function_many
    - ![image2022-9-22 13:53:59.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-22%2013%3A53%3A59.png)
    - ...
    - <https://elixir.bootlin.com/linux/v3.10/source/kernel/smp.c#369>

通过以下命令统计触发中断的触发方: 

stackcount-bpfcc native_send_call_func_*

通过以下命令, 统计 问题机器 (已变成master) 和 一台正常机器 (同业务的slave) 的相关中断触发数量:

  - trace-bpfcc native_send_call_func_single_ipi
  - trace-bpfcc native_send_call_func_ipi

处理命令: 

```
cat /Users/tachikoma/Downloads/113-15-native_send_call_func_single_ipi.txt | sort | awk '{print $1,$3}' | uniq -c | less | sort -nr | head -n 5
```

| native_send_call_func_single_ipi | native_send_call_func_ipi |  |
| --- | --- | --- |
| 13.44 (问题机器) | 1001 201847 mysqld |  | 925 200306 mysqld 705 179304 ustats 519 2568 titanagent
| 13 1 systemd| 9202 201847 mysqld 7125 200306 mysqld 432 179304 ustats 236 2568 titanagent 54 179374 uguard-agent
| 113.15 (正常机器)| 92 2524 ustats 62 126510 mysqld 23 1 systemd 20 13436 easy_collector
| 10 2509 uguard-agent| 6891 126510 mysqld 1720 2655 mysqld 1459 2524 ustats 103 82961 urman-agent 61 2509 uguard-agent
  
通过stackcount 观察一个MySQL的CAL中断来源: 

```
stacktrace -sv -p 201847 native_send_call_func_single_ipi
stacktrace -sv -p 201847 native_send_call_func_ipi
``` 

[13-44-201874-native_send_call_func_ipi.txt](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/13-44-201874-native_send_call_func_ipi.txt)

[13-44-201874-native_send_call_func_single_ipi.txt](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/13-44-201874-native_send_call_func_single_ipi.txt)

大部分堆栈类似于: 

```
  ffffffff9fc59091 native_send_call_func_single_ipi+0x1
  ffffffff9fd1707f smp_call_function_single+0x5f
  ffffffff9fd1762b smp_call_function_many+0x22b
  ffffffff9fc7e8f8 native_flush_tlb_others+0xb8
  ffffffff9fc7e96b flush_tlb_mm_range+0x6b
  ffffffff9fdfe1dd change_protection+0x5cd
  ffffffff9fe1a80b change_prot_numa+0x1b
  ffffffff9fce0952 task_numa_work+0x212
  ffffffff9fcc31cb task_work_run+0xbb
  ffffffff9fc2cc65 do_notify_resume+0xa5
  ffffffffa039322f int_signal+0x12
  7f19af04ac5b     [unknown]
    1467
 
 
  ffffffff9fc59191 native_send_call_func_ipi+0x1
  ffffffff9fc7e8f8 native_flush_tlb_others+0xb8
  ffffffff9fc7e96b flush_tlb_mm_range+0x6b
  ffffffff9fdfe1dd change_protection+0x5cd
  ffffffff9fe1a80b change_prot_numa+0x1b
  ffffffff9fce0952 task_numa_work+0x212
  ffffffff9fcc31cb task_work_run+0xbb
  ffffffff9fc2cc65 do_notify_resume+0xa5
  ffffffffa038957c retint_signal+0x48
  d98f0e           [unknown]
  ce0b32           [unknown]
  d0edf3           [unknown]
  d54b32           [unknown]
  d54dcf           [unknown]
  d15823           [unknown]
  d1909e           [unknown]
  d1af7d           [unknown]
  d1c19a           [unknown]
  d1d044           [unknown]
  ded7ac           [unknown]
  f707b4           [unknown]
  7f19af043ea5     [unknown]
    633
``` 

task_numa_work 为 numa balancing 机制工作, 对NUMA中要迁移的内存块:

  - 标记为缺失内存块 ( change_prot_numa 将其标记为 inaccessible) 
    - ![image2022-9-24 23:53:13.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-24%2023%3A53%3A13.png)
  - 在下次访问时, 触发fault, 将内存块迁移到合适的NUMA区中

标记的过程中:

  - 大页或者普通页的页表, 都会触发 flush_tlb_mm_range
    - ![image2022-9-25 0:5:12.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-9-25%200%3A5%3A12.png)

smp_call_function_many中: 

  - Fast path会调用 smp_call_function_single
  - 其他情况, 会调用 arch_send_call_function_ipi_mask

numa balancing的工作机制: 

  - <https://sysctl-explorer.net/kernel/numa_balancing_scan_period_min_ms,%20numa_balancing_scan_delay_ms,/>
    - numa_balancing_scan_period_min_ms 扫描间隔的最小值
    - numa_balancing_scan_period_max_ms 扫描间隔的最大值
    - 扫描间隔是自适应的, 需要调整的页越多, 扫描应越频繁
    - numa_balancing_scan_delay_ms 第一次扫描的间隔
    - numa_balancing_scan_size_mb 一次扫描的内存大小
    - numa_balancing_settle_count: 每经过X个扫描间隔, 扫描任务将推送给 非最优 节点, 这样不会让 最优节点 过载

线索: 与NUMA/PTE/THP有关的patch: [https://git.froggi.es/mirrors/linux/commit/0255d4918480?style=unified&whitespace=ignore-eol](<https://git.froggi.es/mirrors/linux/commit/0255d4918480?style=unified&whitespace=ignore-eol>)

线索: 找到NUMA balancing扫描的逻辑: <https://blog.csdn.net/faxiang1230/article/details/123709414>

  - __handle_mm_fault
    - do_pmd_numa_page
      - numa_migrate_prep
        - 统计 NUMA_HINT_FAULTS
        - 统计 NUMA_HINT_FAULTS_LOCAL
        - mpol_misplaced: 检查当前页是否符合 MPOL 内存策略 (INTERLEAVE / PREFERRED / BIND)
        - 返回值: 根据MPOL策略, 当前页应当分配到 NUMA node 的ID
      - migrate_misplaced_page
        - numamigrate_isolate_page: 将page从LRU中分离
        - migrate_pages: 迁移page
        - 统计 NUMA_PAGE_MIGRATE
      - task_numa_fault
        - 修订 扫描 周期
        - ![image2022-10-2 16:13:8.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-10-2%2016%3A13%3A8.png)

页表结构: <https://blog.csdn.net/yrj/article/details/2508785>

  - ![image2022-10-1 22:16:3.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-10-1%2022%3A16%3A3.png)
  - PGD/PUD/PMD/PTE 四级页表  

    - Page Global Directory (PGD)

    - Page Upper Directory (PUD) 

    - Page Middle Directory (PMD)

    - Page Table (PTE)

# 采集现场vmstat和dstat

[附件: 13-44-20221002-vmstat.xlsx] [附件: 13-44-20221002-213747-dstat.csv] 

其中: 

  - dstat中的CAL已经减小为0, TLB比之前采集的情况更高
  - 并且: numastat 显示数据分布更加均衡 (长期numa balancing的结果?)
    - ![image2022-10-2 22:1:58.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-10-2%2022%3A1%3A58.png)

对于/proc/interrupts 和 dstat 的CAL数值解释

  - dstat的数据来源于 /proc/interrupts
  - /proc/interrupts中的CAL计算: 
    - 在3.10内核中: CAL = cal - tlb
      - <https://elixir.bootlin.com/linux/v3.10.108/source/arch/x86/kernel/irq.c#L95>
      - ![image2022-10-4 17:24:42.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-10-4%2017%3A24%3A42.png)
    - 在4.20内核中: CAL = cal, 不需去除tlb
      - <https://elixir.bootlin.com/linux/v4.20.17/source/arch/x86/kernel/irq.c>
      - ![image2022-10-4 17:25:39.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-10-4%2017%3A25%3A39.png)

TLB的统计值来源: 

  - ![image2022-10-4 17:29:2.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-10-4%2017%3A29%3A2.png)
  - flush_tlb_func
    - native_flush_tlb_others
      - flush_tlb_others

为什么CAL会比TLB高得多: 

  - 检查stackcount的结果, 大量调用native_flush_tlb_others造成CAL
  - 查找何种情况, 在调用native_flush_tlb_others时, 会增加CAL, 但不会增加TLB
  - 响应IPI时, 才会增加CAL
    - ![image2022-10-5 0:38:1.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-10-5%200%3A38%3A1.png)
    - generic_smp_call_function_sngle_interrupt中, 会调用目标函数: 
      - ![image2022-10-5 0:39:41.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-10-5%200%3A39%3A41.png)
    - 在native_flush_tlb_others中指定了目标函数: 
      - ![image2022-10-5 0:40:32.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-10-5%200%3A40%3A32.png)
    - flush_tlb_func会增加 TLB:
      - ![image2022-10-5 0:41:6.png](/assets/01KJBYXQHSCD3DVCRPTGQVN88W/image2022-10-5%200%3A41%3A6.png)
