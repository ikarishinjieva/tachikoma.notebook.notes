---
title: 20220808 - 百胜slave延迟, 关于内存页表的相关诊断
confluence_page_id: 1933590
created_at: 2022-08-08T03:33:28+00:00
updated_at: 2022-08-17T08:39:54+00:00
---

# 现象

  - 百胜slave发生延迟
    - ![image2022-8-8 11:28:49.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-8%2011%3A28%3A49.png)
  - 会有uproxy的空闲连接连接到这台slave上, 一旦空闲连接被关闭, 延迟很快消失

# 发现

  - 出问题的slave, 未调整过cstate, 无论是否出现slave延时, 页表dcycle的消耗在10% 左右:   

    - ![image2022-8-8 11:30:48.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-8%2011%3A30%3A48.png)
  - 其他slave, 都调整过cstate (intel_idle.max_cstate=0), 页表dcycle的消耗在5%左右:   

    - ![image2022-8-8 11:31:52.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-8%2011%3A31%3A52.png)
    - 也可能是因为 正常的slave 会乘载读写分离的读流量, 导致页表压力不同
  - 出问题的从, IPC偏低: 
    - ![image2022-8-8 11:32:55.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-8%2011%3A32%3A55.png)
  - 正常的从, IPC正常: 
    - ![image2022-8-8 11:33:9.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-8%2011%3A33%3A9.png)

将 出问题的slave 的空闲连接关掉 (连接数从 12.7k 下降到 200+), 观察到dcycles占比下降, 无延迟: 

![image2022-8-9 13:1:52.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-9%2013%3A1%3A52.png)

有链接有延时时, IPC下降到0.2

# 需要解释perf events

  - <https://perfmon-events.intel.com/ivytown.html>
    - 可以找到对应 event 的编号和掩码
  - <https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html>
    - ![image2022-8-9 16:29:13.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-9%2016%3A29%3A13.png)
    - PEBS = Processor Event Based Sampling (PEBS) Events
  - <https://insujang.github.io/2017-04-03/intel-sgx-protection-mechanism/>
    - TLB和PMH的关系

    - When a virtual address is not contained in a core’s TLB, the Page Miss Handler (PMH) performs a page walk to translate the virtual address, and the result is stored in the TLB.

  - <https://en.wikipedia.org/wiki/Translation_lookaside_buffer>
    - TLB-miss handling
      - ![image2022-8-10 14:26:25.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-10%2014%3A26%3A25.png)

# NUMA 和 THP 配合的性能下降

<https://people.ece.ubc.ca/sasha/papers/atc14-final132.pdf>

![image2022-8-10 19:16:47.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-10%2019%3A16%3A47.png)

NUMA和THP配合, 不同压力下, 会导致性能上升或下降. 上表 测试了五种压力, 两台压力 (A/B)

性能被拆分成一下四个维度测量: 

  - page fault handler的耗时 (毫秒)
  - L2 未命中, 导致的page table walk (百分比)
    - DTLB_LOAD_MISSES.MISS_CAUSES_A_WALK / cycles

```
perf stat -e cycles \
-e cycles
-e cpu/event=0x08,umask=0x01,name=DTLB_LOAD_MISSES.MISS_CAUSES_A_WALK/ \
-e cpu/event=0x08,umask=0x10,cmask=0x1,name=DTLB_LOAD_MISSES.WALK_ACTIVE/ \
-e cpu/event=0x08,umask=0x10,name=DTLB_LOAD_MISSES.WALK_PENDING/ \
-e cpu/event=0x08,umask=0x20,name=DTLB_LOAD_MISSES.STLB_HIT/ \
-e cpu/event=0x08,umask=0x0e,name=DTLB_LOAD_MISSES.WALK_COMPLETED/ \
-e cpu/event=0x08,umask=0x08,name=DTLB_LOAD_MISSES.WALK_COMPLETED_1G	/ \
-e cpu/event=0x08,umask=0x04,name=DTLB_LOAD_MISSES.WALK_COMPLETED_2M_4M	/ \
-e cpu/event=0x08,umask=0x02,name=DTLB_LOAD_MISSES.WALK_COMPLETED_4K	/ \
-e cpu/event=0x49,umask=0x01,name=DTLB_STORE_MISSES.MISS_CAUSES_A_WALK		/ \
-e cpu/event=0x49,umask=0x10,name=DTLB_STORE_MISSES.WALK_PENDING		/ \
-e cpu/event=0x49,umask=0x10,cmask=0x1,name=DTLB_STORE_MISSES.WALK_ACTIVE		/ \
-e cpu/event=0x49,umask=0x20,name=DTLB_STORE_MISSES.STLB_HIT		/ \
-e cpu/event=0x49,umask=0x0e,name=DTLB_STORE_MISSES.WALK_COMPLETED			/ \
-e cpu/event=0x49,umask=0x08,name=DTLB_STORE_MISSES.WALK_COMPLETED_1G			/ \
-e cpu/event=0x49,umask=0x04,name=DTLB_STORE_MISSES.WALK_COMPLETED_2M_4M			/ \
-e cpu/event=0x49,umask=0x02,name=DTLB_STORE_MISSES.WALK_COMPLETED_4K			/ \
-a -I 1000
``` 
  - Local access ratio: 访问本地Node的访问比例
    - 测定方法1: `perf stat -e node-load-misses,node-loads,node-prefetch-misses,node-prefetches,node-store-misses,node-stores -p $(pgrep mysqld$) -- sleep ``5`
    - `测定方法2: numatop`
  - Imbalance (Traffic imbalance): 不同Node的访问比重失衡
    - `测定方法: `pcm-numa

# 参考

  - c-state, p-state的区别: <https://metebalci.com/blog/a-minimum-complete-tutorial-of-cpu-power-management-c-states-and-p-states/>

# 猜测原因

  - <https://www.anycodings.com/1questions/20655/why-does-my-intel-skylake-kaby-lake-cpu-incur-a-mysterious-factor-3-slowdown-in-a-simple-hash-table-implementation>
    - 文章指出: Skylake架构 要保证Page Table coherence, TLB不命中的Load和address-unknown的store, 不能并发执行, 导致memory速率降低
    - 在测试代码的循环中, 循环N+1的load, 需要扫页表 (page table walk), 要等待循环N的store完成才能进行
      - dtlb_load_misses.miss_causes_a_walk 是触发walk的次数. 触发walk后, 由于walk发现 循环N的store 存在, 所以需要等待下一轮的walk. 因此会比 dtlb_load_misses.walk_completed 大一倍
        - ![image2022-8-17 11:54:33.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-17%2011%3A54%3A33.png)
      - 在百胜的场景中, 未观察到这一现象
    - 测定MLP (Memory-level parallelism)
      - MLP = l1d_pend_miss.pending / l1d_pend_miss.pending_cycles
        - l1d_pend_miss.pending: L1D 的 miss 数量
        - l1d_pend_miss.pending_cycles: L1D的 miss周期
        - 两者相除, 近似于 内存访问的并发度
      - 在百胜的场景中, 未观察到异常: 
        - ![image2022-8-17 16:39:33.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-17%2016%3A39%3A33.png)
        - ![image2022-8-17 16:39:39.png](/assets/01KJBYKEEAEBE19H8NX3SFJXR3/image2022-8-17%2016%3A39%3A39.png)
        - 239.26 为故障节点, 带有主从延迟
  - 学习 Page Table coherence
    - <https://blog.stuffedcow.net/2015/08/pagewalk-coherence/>
      - Page Table 的基础知识, 介绍得很好
      - coherence分类
        - TLB coherence (页表 项改变时, TLB是否跟着改变)
          - x86不需要保障TLB的一致性: In x86 (and all other architectures I can think of), the TLB is not coherent
        - pagewalk coherence (Page walk(页表扫描) 与 页表更新 之间的一致性)
          - 保持pagewalk一致性的三种设计
            - Non-coherent pagewalks: 不保持一致性
            - coherent pagewalks done non-speculatively: 一刀切, 等待之前的指令全部执行完, 再进行pagewalk
            - coherent pagewalks done speculatively with detect-and-retry: 当发现page table改变, 重启pagewalk
