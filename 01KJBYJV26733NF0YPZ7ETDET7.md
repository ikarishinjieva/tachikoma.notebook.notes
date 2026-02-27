---
title: 20220623 - Top-down Microarchitecture Analysis 尝试
confluence_page_id: 1933348
created_at: 2022-06-23T05:34:29+00:00
updated_at: 2022-07-06T07:54:37+00:00
---

# 尝试 Intel Vtune

  - <https://www.intel.com/content/www/us/en/develop/documentation/vtune-help/top.html>
  - 安装, 并打开demo 结果, 截图如下: 
  - 但同步过Mac UI, 远程调试Linux中的程序 未成功: 连接ssh未成功, 但不知具体报错
  - ![image2022-6-22 19:1:38.png](/assets/01KJBYJV26733NF0YPZ7ETDET7/image2022-6-22%2019%3A1%3A38.png)
  - ![image2022-6-22 19:2:29.png](/assets/01KJBYJV26733NF0YPZ7ETDET7/image2022-6-22%2019%3A2%3A29.png)
  - ![image2022-6-22 19:2:51.png](/assets/01KJBYJV26733NF0YPZ7ETDET7/image2022-6-22%2019%3A2%3A51.png)

  - 对里面每一个指标的解释: <https://www.intel.com/content/www/us/en/develop/documentation/vtune-help/top/reference/cpu-metrics-reference.html#cpu-metrics-reference_CLOCKTICKS-VS-PIPELINE-SLOTS-BASED-METRICS>
  - 对 perf event 的解释
    - <https://www.cism.ucl.ac.be/Services/Formations/ICS/ics_2013.0.028/vtune_amplifier_xe/documentation/en/help/reference/index.htm#pmbk/events/about_precise_event_based_sampling_performance_tuning_events.html>

# 研究Vtune是如何计算 Memroy bound的

获取 r001ue 样例集中的multiple1函数的采样数据做对比

bottom-up 的分析数据: 

```
Microarchitecture Usage:	4.7% of Pipeline Slots
    Retiring:	4.7%
    Front-End Bound:	9.3%
    Bad Speculation:	0.4%
    Back-End Bound:	85.5%
    	Memory Bound:	81.7%
		    L1 Bound:	0.0%
		    L2 Bound:	0.0%
		    L3 Bound:	1.1%
	    DRAM Bound:	91.6%
		    Memory Bandwidth:	87.1%
		    Memory Latency:	11.6%
	    Store Bound:	0.0%
    Core Bound:	3.8%
    	Divider:	0.0%
	    Port Utilization:	4.1%
    		Cycles of 0 Ports Utilized:	82.7%
		    Cycles of 1 Port Utilized:	8.1%
		    Cycles of 2 Ports Utilized:	4.6%
		    Cycles of 3+ Ports Utilized:	3.7%
		    Vector Capacity Usage (FPU):	25.0%
``` 

Event count数据: 

```
Hardware Events
    Hardware Event Type	Hardware Event Count
    CPU_CLK_UNHALTED.ONE_THREAD_ACTIVE	33,293,161
    CPU_CLK_UNHALTED.REF_TSC	709,905,000,000
    CPU_CLK_UNHALTED.REF_XCLK	4,004,789,793
    CPU_CLK_UNHALTED.THREAD	745,248,000,000
    CPU_CLK_UNHALTED.THREAD_P	745,938,823,088
    CYCLE_ACTIVITY.STALLS_L1D_MISS	690,684,088,284
    CYCLE_ACTIVITY.STALLS_L2_MISS	689,751,714,792
    CYCLE_ACTIVITY.STALLS_L3_MISS	681,690,359,095
    CYCLE_ACTIVITY.STALLS_MEM_ANY	654,256,732,603
    DTLB_LOAD_MISSES.STLB_HIT	650,391,304
    DTLB_LOAD_MISSES.WALK_ACTIVE	278,215,782,205
    DTLB_STORE_MISSES.WALK_ACTIVE	175,546,747
    EXE_ACTIVITY.1_PORTS_UTIL	29,623,288,337
    EXE_ACTIVITY.2_PORTS_UTIL	14,262,876,862
    EXE_ACTIVITY.EXE_BOUND_0_PORTS	650,391,304
    FP_ARITH_INST_RETIRED.SCALAR_DOUBLE	15,967,808,148
    FRONTEND_RETIRED.DSB_MISS_PS	159,926,588
    FRONTEND_RETIRED.LATENCY_GE_1	78,807,970
    FRONTEND_RETIRED.LATENCY_GE_16_PS	32,521,792
    FRONTEND_RETIRED.LATENCY_GE_2_BUBBLES_GE_1_PS	49,998,690
    FRONTEND_RETIRED.LATENCY_GE_8_PS	31,273,537
    IDQ.ALL_DSB_CYCLES_4_UOPS	12,639,568,515
    IDQ.ALL_DSB_CYCLES_ANY_UOPS	13,300,295,345
    IDQ.ALL_MITE_CYCLES_4_UOPS	650,391,304
    IDQ.DSB_UOPS	78,054,342,696
    IDQ.MS_UOPS	625,427,910
    IDQ_UOPS_NOT_DELIVERED.CORE	140,393,840,957
    IDQ_UOPS_NOT_DELIVERED.CYCLES_0_UOPS_DELIV.CORE	33,802,373,221
    INST_RETIRED.ANY	68,775,000,000
    INST_RETIRED.PREC_DIST	111,096,679,936
    INT_MISC.CLEAR_RESTEER_CYCLES	16,092,469,957
    L1D_PEND_MISS.FB_FULL:cmask=1	91,405,664,172
    L1D_PEND_MISS.PENDING	4,298,466,894,753
    LD_BLOCKS_PARTIAL.ADDRESS_ALIAS	111,221,260
    MACHINE_CLEARS.COUNT	32,520,492
    MEM_INST_RETIRED.ALL_STORES_PS	17,910,079,450
    MEM_INST_RETIRED.STLB_MISS_LOADS_PS	11,306,019,845
    MEM_LOAD_L3_HIT_RETIRED.XSNP_HIT_PS	192,615,824
    MEM_LOAD_L3_HIT_RETIRED.XSNP_MISS_PS	19,218,163
    MEM_LOAD_RETIRED.FB_HIT_PS	2,081,850,752
    MEM_LOAD_RETIRED.L1_HIT_PS	15,141,838,864
    MEM_LOAD_RETIRED.L1_MISS_PS	11,516,834,575
    MEM_LOAD_RETIRED.L2_HIT_PS	1,157,378,937
    MEM_LOAD_RETIRED.L3_HIT_PS	942,962,807
    MEM_LOAD_RETIRED.L3_MISS_PS	10,178,295,354
    OFFCORE_REQUESTS_BUFFER.SQ_FULL	2,646,125,350
    OFFCORE_REQUESTS_OUTSTANDING.ALL_DATA_RD:cmask=4	649,384,639,917
    OFFCORE_REQUESTS_OUTSTANDING.CYCLES_WITH_DATA_RD	735,963,449,040
    OFFCORE_REQUESTS_OUTSTANDING.CYCLES_WITH_DEMAND_RFO	343,720,588
    UOPS_DISPATCHED_PORT.PORT_0	56,953,244,043
    UOPS_DISPATCHED_PORT.PORT_1	53,213,480,791
    UOPS_DISPATCHED_PORT.PORT_2	11,585,807,837
    UOPS_DISPATCHED_PORT.PORT_3	13,238,233,265
    UOPS_DISPATCHED_PORT.PORT_4	7,129,172,901
    UOPS_DISPATCHED_PORT.PORT_5	14,193,310,978
    UOPS_DISPATCHED_PORT.PORT_6	19,392,783,654
    UOPS_EXECUTED.CORE_CYCLES_GE_1	122,991,992,179
    UOPS_EXECUTED.CORE_CYCLES_GE_2	61,890,320,100
    UOPS_EXECUTED.CORE_CYCLES_GE_3	27,596,663,192
    UOPS_EXECUTED.CORE_CYCLES_NONE	621,645,978,437
    UOPS_EXECUTED.THREAD	90,948,520,844
    UOPS_ISSUED.ANY	77,036,736,224
    UOPS_RETIRED.RETIRE_SLOTS	70,297,108,026

``` 

# 整理相关概念

  - TMA (Top-down Microarchitecture Analysis): 
    - ![image](https://pic1.zhimg.com/v2-b7fa8d904f4c092402e501710dfb8700_1440w.jpg?source=172ae18b)
    - 让CPU黑盒不再黑——【TMA_自顶向下的CPU架构性能瓶颈分析方法】（一）What & Why: <https://zhuanlan.zhihu.com/p/60569271>

      - 与 VTUNE 类似实现了TMA的工具: [andikleen/pmu-tools](<https://link.zhihu.com/?target=https%3A//github.com/andikleen/pmu-tools>)

      - 在最“好”的情况下，Retiring的比重为100%，其余比重为0%，即其余三个分类可以理解为三类导致CPU效率不高的分类。
    - 让CPU黑盒不再黑——【TMA_自顶向下的CPU架构性能瓶颈分析方法】（二）How: <https://zhuanlan.zhihu.com/p/60940902>

      - 后端负责接收前端的指令，并且乱序执行，顺序Retire

      - ![image](https://pic4.zhimg.com/80/v2-fdee4789a17b29a936d2ba32022b436b_1440w.jpg)
    - 让CPU黑盒不再黑——【TMA_自顶向下的CPU架构性能瓶颈分析方法】（三）Frontend: <https://zhuanlan.zhihu.com/p/61015720>
      - 公式均来源于TMAM 3.5中的skl_client_ratios.py文件，该文件主要存放的是Skylake的TMA比例系数和公式: <https://github.com/andikleen/pmu-tools/blob/master/skl_client_ratios.py>
      - 分析了Frontend的各性能项
    - 让CPU黑盒不再黑——【TMA_自顶向下的CPU架构性能瓶颈分析方法】（四）Speculation: <https://zhuanlan.zhihu.com/p/64529137>

      - 分析了Speculation的各性能项
    - 让CPU黑盒不再黑——【TMA_自顶向下的CPU架构性能瓶颈分析方法】（五）Retiring: <https://zhuanlan.zhihu.com/p/64576459>

      -         - 分析了Retiring的各性能项

  - uops: 
    - 【uOps哲学三问】我是谁？——带你梳理x86微架构: <https://zhuanlan.zhihu.com/p/509264784>

    - 【uOps哲学三问】我来自哪里？——带你梳理x86微架构: <https://zhuanlan.zhihu.com/p/511035787>

# 实验1

沿用 [20220612 - numa blancing] 的实验: 测试 同NUMA和跨NUMA的MySQL访问: 

同NUMA: 

```
[root@R820-04 pmu-tools-master]# ./toplev -l10 --pid $(pgrep mysqld$) -S sleep 5
5 events not counted
warning: no idle detection because no cycle event found
# 4.4-full-perf on Intel(R) Xeon(R) CPU E5-4620 0 @ 2.20GHz [jkt/sandybridge]
FE             Frontend_Bound                                % Slots                      36.9   [ 0.0%]
BE             Backend_Bound                                 % Slots                      55.7   [ 0.0%]
FE             Frontend_Bound.Fetch_Latency                  % Slots                      26.2   [ 0.0%]
BE/Mem         Backend_Bound.Memory_Bound                    % Slots                      32.2   [ 0.0%]<==
	This metric represents fraction of slots the Memory
	subsystem within the Backend was a bottleneck...
BE/Core        Backend_Bound.Core_Bound                      % Slots                      23.6   [ 0.0%]
FE             Frontend_Bound.Fetch_Latency.ITLB_Misses      % Clocks                      5.1   [ 0.0%]
	This metric represents fraction of cycles the CPU was
	stalled due to Instruction TLB (ITLB) misses...
	Sampling events:  itlb_misses.walk_completed
FE             Frontend_Bound.Fetch_Latency.Branch_Resteers  % Clocks_est                 20.8   [ 0.0%]
	This metric represents fraction of cycles the CPU was
	stalled due to Branch Resteers...
	Sampling events:  br_misp_retired.all_branches
BE/Core        Backend_Bound.Core_Bound.Ports_Utilization    % Clocks                     21.9   [ 0.0%]
	This metric estimates fraction of cycles the CPU performance
	was potentially limited due to Core computation issues (non
	divider-related)...
1 results not referenced:  8
3 nodes had zero counts: DRAM_Bound L3_Bound MEM_Bandwidth
Mismeasured (out of bound values): FP_Arith X87_Use
Run toplev --describe Memory_Bound^ to get more information on bottleneck
``` 

跨NUMA:

```
[root@R820-04 pmu-tools-master]# ./toplev -l10 --pid $(pgrep mysqld$) -S sleep 5
5 events not counted
warning: no idle detection because no cycle event found
# 4.4-full-perf on Intel(R) Xeon(R) CPU E5-4620 0 @ 2.20GHz [jkt/sandybridge]
FE             Frontend_Bound                                % Slots                      20.7   [ 0.0%]
BE             Backend_Bound                                 % Slots                      73.2   [ 0.0%]
FE             Frontend_Bound.Fetch_Latency                  % Slots                      23.9   [ 0.0%]
BE/Mem         Backend_Bound.Memory_Bound                    % Slots                      45.2   [ 0.0%]<==
	This metric represents fraction of slots the Memory
	subsystem within the Backend was a bottleneck...
BE/Core        Backend_Bound.Core_Bound                      % Slots                      28.0   [ 0.0%]
FE             Frontend_Bound.Fetch_Latency.Branch_Resteers  % Clocks_est                 17.5   [ 0.0%]
	This metric represents fraction of cycles the CPU was
	stalled due to Branch Resteers...
	Sampling events:  br_misp_retired.all_branches
BE/Core        Backend_Bound.Core_Bound.Ports_Utilization    % Clocks                     19.5   [ 0.0%]
	This metric estimates fraction of cycles the CPU performance
	was potentially limited due to Core computation issues (non
	divider-related)...
1 results not referenced:  8
3 nodes had zero counts: DRAM_Bound L3_Bound MEM_Bandwidth
Mismeasured (out of bound values): FP_Arith X87_Use
Run toplev --describe Memory_Bound^ to get more information on bottleneck
Add --run-sample to find locations
``` 
    
    
    Backend_Bound.Memory_Bound 在 跨NUMA访问时升高, 偏差 15% 左右, 跟MySQL的性能偏差比例 类似

针对 10.186.17.104 的监控项算式: <https://github.com/andikleen/pmu-tools/blob/7a422d6e807a523be9fcb78464505e10b21ab2c9/jkt_server_ratios.py>

TODO: 需要解释: 

  1. Memory bound 的计算如何得来
  2. 为什么不能再往下一层的原因进行分析

# 有用的文档

<https://www.cs.utexas.edu/~pingali/CS377P/2019sp/lectures/vtune-cache-jackson.pdf>
