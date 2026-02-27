---
title: 20220612 - numa blancing
confluence_page_id: 1737242
created_at: 2022-06-12T02:06:00+00:00
updated_at: 2022-08-24T09:27:23+00:00
---

# 参考

  - <https://support.oracle.com/knowledge/Oracle%20Linux%20and%20Virtualization/2749259_1.html>
    - NUMA balancing的工作模式:

      - 1\. A task scanner periodically scans a portion of a task's address space and marks the memory to force a page fault when the data is next accessed.

        - 周期性运行scanner, 标记一部分内存, 当这部分内存数据被访问时, 触发page fualt
      - 2\. The next access to the data will result in a NUMA Hinting Fault. Based on this fault, the data can be migrated to a memory node associated with the task accessing the memory.

        - 这部分内存数据被访问时, 触发 NUMA Hinting Fault. 触发page fault时, 数据可发生迁移
      - 3\. To keep a task, the CPU it is using and the memory it is accessing together, the scheduler groups tasks that share data.

    - It is due to this induced page fault for page migration that causes high %iowait.

      - page fault会引起高 iowait
    - /proc/vmstat 中的统计值
      - numa_pte_updates

        - The amount of base pages that were marked for NUMA hinting faults.

      - numa_huge_pte_updates

        - The amount of transparent huge pages that were marked for NUMA hinting faults. In combination with numa_pte_updates the total address space that was marked can be calculated.

      - numa_hint_faults

        - Records how many NUMA hinting faults were trapped

      - numa_hint_faults_local
        - Shows how many of the hinting faults were to local nodes. In combination with numa_hint_faults, the percentage of local versus remote faults can be calculated. A high percentage of local hinting faults indicates that the workload is closer to being converged.
      - numa_pages_migrated
        - Records how many pages were migrated because they were misplaced. As migration is a copying operation, it contributes the largest part of the overhead created by NUMA balancing
  - <https://docs.kernel.org/admin-guide/numastat.html>  

    - /sys/devices/system/node/node*/numastat 有类似vmstat的统计值

| - numa_hit | A process wanted to allocate memory from this node, and succeeded. |
| --- | --- |
| numa_miss | A process wanted to allocate memory from another node, but ended up with memory from this node. |
| numa_foreign | A process wanted to allocate on this node, but ended up with memory from another node. |
| local_node | A process ran on this node’s CPU, and got memory from this node. |
| other_node | A process ran on a different node’s CPU and got memory from this node. |
| interleave_hit | Interleaving wanted to allocate from this node and succeeded. | - <https://blog.jcole.us/2010/09/28/mysql-swap-insanity-and-the-numa-architecture/> - /proc/pid/numa_maps - 样例输出:

```
2aaaaad3e000 default anon=13240527 dirty=13223315 
  swapcache=3440324 active=13202235 N0=7865429 N1=5375098
 
---
2aaaaad3e000 — The virtual address of the memory region. Ignore this other than the fact that it’s a unique ID for this piece of memory.
default — The NUMA policy in use for this region.
anon=number — The number of anonymous pages mapped.
dirty=number — The number of pages that are dirty because they have been modified. Generally memory allocated only within a single process is always going to be used, and thus dirty, but if a process forks it may have many copy-on-write pages mapped that are not dirty.
swapcache=number — The number of pages swapped out but unmodified since they were swapped out, and thus they are ready to be freed if needed, but are still in memory at the moment.
active=number — The number of pages on the “active list”; if this field is shown, some memory is inactive (anon minus active) which means it may be paged out by the swapper soon.
N0=number and N1=number — The number of pages allocated on Node 0 and Node 1, respectively.
``` 
  - <https://plantegg.github.io/2021/05/14/%E5%8D%81%E5%B9%B4%E5%90%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%98%E6%98%AF%E4%B8%8D%E6%95%A2%E6%8B%A5%E6%8A%B1NUMA/>
    - Intel的处理器NUMA调度图  

      - ![image](https://plantegg.github.io/images/951413iMgBlog/1623830161880-c4c74f4d-785e-4274-a579-5d1aa8b5e990.png)
      - ![image](https://plantegg.github.io/images/951413iMgBlog/03-05-Broadwell_HCC_Architecture.svg)
      - 跨socket (红环) 的延迟 (跨NUMA延迟), 可以通过工具测量: 
        - <https://www.intel.com/content/www/us/en/developer/articles/tool/intelr-memory-latency-checker.html>

  - pcm 工具, 用于观察 跨NUMA的内存访问
    - <https://www.intel.com/content/www/us/en/developer/articles/technical/performance-counter-monitor.html>
    - 安装: [https://software.opensuse.org/download/package?package=pcm&project=home%3Aopcm](<https://software.opensuse.org/download/package?package=pcm&project=home%3Aopcm>)

# 实验1

试验机: 10.186.17.104

关闭 numa_balancing:

```
echo 0 > /proc/sys/kernel/numa_balancing
``` 

切换mysqld的numa配置: 

```
taskset -apc 0,4,8,12,16,20,24,28,32,36,40,44,48,52,56,60 $(pgrep mysqld$) #node 0
taskset -apc 1,5,9,13,17,21,25,29,33,37,41,45,49,53,57,61 $(pgrep mysqld$) #node 1
taskset -apc 2,6,10,14,18,22,26,30,34,38,42,46,50,54,58,62 $(pgrep mysqld$) #node 2
taskset -apc 3,7,11,15,19,23,27,31,35,39,43,47,51,55,59,63 $(pgrep mysqld$) #node 3
``` 

需要使用 -a, 否则taskset只针对线程作用

先重启数据库, 清空bp

查看初始状况: 

```
[root@R820-04 ~]# numastat -p $(pgrep mysqld$)

Per-node process memory usage (in MBs) for PID 60449 (mysqld)
                           Node 0          Node 1          Node 2
                  --------------- --------------- ---------------
Huge                         0.00            0.00            0.00
Heap                         0.00            0.00            0.00
Stack                        0.00            0.00            0.00
Private                    224.73         2502.71          225.29
----------------  --------------- --------------- ---------------
Total                      224.73         2502.71          225.29

                           Node 3           Total
                  --------------- ---------------
Huge                         0.00            0.00
Heap                        34.13           34.13
Stack                        0.06            0.06
Private                    418.89         3371.62
----------------  --------------- ---------------
Total                      453.08         3405.81
``` 

### 同NUMA访问

然后将cpu绑到node 3上

```
taskset -apc 3,7,11,15,19,23,27,31,35,39,43,47,51,55,59,63 $(pgrep mysqld$) #node 3
``` 

使用sysbench压力: 

```
sysbench /usr/share/sysbench/oltp_point_select.lua --mysql-host=127.0.0.1 --mysql-port=8022 --mysql-user=msandbox --mysql-password=msandbox --mysql-db=test --tables=64 --table-size=1000000 --report-interval=3 --threads=16 --time=300 run
``` 

检查buffer pool分布: 

```
[root@R820-04 ~]# numastat -p $(pgrep mysqld$)

Per-node process memory usage (in MBs) for PID 60449 (mysqld)
                           Node 0          Node 1          Node 2
                  --------------- --------------- ---------------
Huge                         0.00            0.00            0.00
Heap                         0.00            0.00            0.00
Stack                        0.00            0.00            0.00
Private                    224.73         2503.08          225.29
----------------  --------------- --------------- ---------------
Total                      224.73         2503.08          225.29

                           Node 3           Total
                  --------------- ---------------
Huge                         0.00            0.00
Heap                        34.13           34.13
Stack                        0.06            0.06
Private                   4586.73         7539.83
----------------  --------------- ---------------
Total                     4620.92         7574.02
``` 

可以看到, bp都建立在了node 3

sysbench 结果大概在 71388.09

观察 perf stat -p {pid}

```
[root@R820-04 ~]# perf stat -p $(pgrep mysqld$) -- sleep 5

 Performance counter stats for process id '60449':

         56,144.16 msec task-clock                #   11.206 CPUs utilized
           333,181      context-switches          #    0.006 M/sec
            14,393      cpu-migrations            #    0.256 K/sec
             1,742      page-faults               #    0.031 K/sec
   123,768,717,104      cycles                    #    2.204 GHz                      (83.33%)
   111,596,549,008      stalled-cycles-frontend   #   90.17% frontend cycles idle     (83.32%)
   100,393,790,112      stalled-cycles-backend    #   81.11% backend cycles idle      (66.89%)
    22,831,787,703      instructions              #    0.18  insn per cycle
                                                  #    4.89  stalled cycles per insn  (83.36%)
     4,644,223,614      branches                  #   82.720 M/sec                    (83.29%)
       215,285,951      branch-misses             #    4.64% of all branches          (83.23%)

       5.009998946 seconds time elapsed
``` 

使用脚本抓取 vmstat 状态: 

脚本: 

```
rm -f /tmp/1.out
for i in {1..10}; do
cat /proc/vmstat | grep numa_ >> /tmp/1.out
sleep 1
done
``` 

使用脚本抓取 numastat 状态: 

```
[root@R820-04 ~]# cat /tmp/2.sh
rm -f /tmp/2.out
for i in {1..10}; do
grep '' /sys/devices/system/node/node*/numastat >> /tmp/2.out
sleep 1
done
``` 

抓取PCM: 

```
Time elapsed: 5000 ms
Core | IPC  | Instructions | Cycles  |  Local DRAM accesses | Remote DRAM Accesses
   0   0.38        647 M     1715 M       712 K              1010 K
   1   0.34        593 M     1765 M       767 K               889 K
   2   0.35        478 M     1384 M       639 K               735 K
   3   0.20       2250 M       11 G      4581 K              4079 K
   4   0.07       2041 K       28 M      4623                1964
   5   0.32         77 M      242 M       105 K               114 K
   6   0.34         70 M      207 M       133 K                68 K
   7   0.21       2312 M       11 G      4586 K              4175 K
   8   0.27       3234 K       12 M      6089                2312
   9   0.13        854 K     6373 K      2386                1308
  10   0.36       9322 K       25 M        16 K              5991
  11   0.20       2211 M       10 G      4652 K              4188 K
  12   0.14        879 K     6450 K      3479                1533
  13   0.14        833 K     6145 K      2530                1245
  14   0.31       5194 K       16 M        11 K              6464
  15   0.20       2101 M       10 G      4403 K              3748 K
  16   0.20       1369 K     6722 K      2854                1300
  17   0.07        337 K     4808 K      1461                 613
  18   0.33         11 M       36 M        14 K              5901
  19   0.20       2117 M       10 G      4568 K              3974 K
  20   0.13        807 K     6461 K      2364                1547
  21   0.08        425 K     5073 K      1555                 765
  22   0.21       5495 K       26 M        11 K              5328
  23   0.21       2268 M       11 G      4636 K              4030 K
  24   0.16        931 K     5984 K      2191                1156
  25   0.08        406 K     4967 K      1487                 799
  26   0.29       8754 K       29 M        21 K              7530
  27   0.21       2274 M       11 G      4669 K              4112 K
  28   0.18       1258 K     7098 K      3500                2193
  29   0.05        461 K     9339 K      1919                1061
  30   0.23         13 M       60 M        20 K              9747
  31   0.21       2284 M       10 G      4544 K              4249 K
  32   0.01        771 K       73 M      9153                 332
  33   0.01        448 K       82 M       657                 389
  34   0.00        323 K       66 M      3247                 636
  35   0.20       2222 M       11 G      4501 K              3986 K
  36   0.10       2356 K       23 M      3316                1654
  37   0.11       2896 K       26 M      3431                4224
  38   0.18       4791 K       26 M        14 K              3759
  39   0.21       2329 M       11 G      4592 K              4192 K
  40   0.21       1603 K     7645 K      4073                1165
  41   0.07        286 K     4332 K      1600                 757
  42   0.07        471 K     6708 K      6697                1043
  43   0.20       2208 M       10 G      4737 K              4009 K
  44   0.09        419 K     4768 K      2081                 811
  45   0.07        336 K     4606 K      1803                 632
  46   0.22       1475 K     6740 K      5788                1021
  47   0.20       2221 M       11 G      4570 K              4024 K
  48   0.09        411 K     4671 K      1815                 767
  49   0.08        362 K     4557 K      1359                 715
  50   0.02        293 K       11 M      5698                 834
  51   0.20       2082 M       10 G      4492 K              3888 K
  52   0.09        405 K     4724 K      1580                 749
  53   0.08        356 K     4630 K      1378                 712
  54   0.02        221 K       12 M      5004                 732
  55   0.20       2267 M       11 G      4727 K              3988 K
  56   0.08        354 K     4492 K      1652                 733
  57   0.07        293 K     4342 K      1393                 728
  58   0.03        252 K     8410 K      8721                1177
  59   0.21       2317 M       11 G      4744 K              4135 K
  60   0.14        942 K     6519 K      4071                2290
  61   0.09        801 K     8766 K      2946                1453
  62   0.02        454 K       27 M      7975                1546
  63   0.21       2318 M       11 G      4594 K              4250 K
-------------------------------------------------------------------------------------------------------------------
   *   0.21         37 G      181 G        76 M                67 M
``` 

通过perf监控NUMA的命中次数: 

```
[root@R820-04 ~]# perf stat -e node-load-misses,node-loads,node-prefetch-misses,node-prefetches,node-store-misses,node-stores -p $(pgrep mysqld$) -- sleep 5

 Performance counter stats for process id '60449':

         8,684,148      node-load-misses                                              (16.57%)
        37,547,906      node-loads                                                    (16.67%)
           169,069      node-prefetch-misses                                          (16.50%)
         4,624,606      node-prefetches                                               (16.48%)
         2,886,445      node-store-misses                                             (16.61%)
        24,153,969      node-stores                                                   (16.53%)

       5.005530710 seconds time elapsed
``` 

### 跨NUMA访问

然后将cpu绑到node 1上

```
taskset -apc 1,5,9,13,17,21,25,29,33,37,41,45,49,53,57,61 $(pgrep mysqld$) #node 1
``` 

跑sysbench, 结果: 62935.35

再次获取perf状态: 

```
[root@R820-04 ~]# perf stat -p $(pgrep mysqld$) -- sleep 5

 Performance counter stats for process id '60449':

         57,632.20 msec task-clock                #   11.503 CPUs utilized
           288,875      context-switches          #    0.005 M/sec
            16,655      cpu-migrations            #    0.289 K/sec
               819      page-faults               #    0.014 K/sec
   127,265,361,587      cycles                    #    2.208 GHz                      (83.45%)
   116,603,929,920      stalled-cycles-frontend   #   91.62% frontend cycles idle     (83.35%)
   106,962,026,215      stalled-cycles-backend    #   84.05% backend cycles idle      (66.80%)
    19,860,069,668      instructions              #    0.16  insn per cycle
                                                  #    5.87  stalled cycles per insn  (83.40%)
     4,039,814,707      branches                  #   70.096 M/sec                    (83.25%)
       187,279,053      branch-misses             #    4.64% of all branches          (83.20%)

       5.010192299 seconds time elapsed
``` 

vmstat 和 numastat 都没有太大变化, 原因是: 两者统计的都是 跨NUMA内存分配 的程度, 并不会对 跨NUMA的内存访问

抓取PCM: 

```
> pcm-numa 5
...
 
 Time elapsed: 5001 ms
Core | IPC  | Instructions | Cycles  |  Local DRAM accesses | Remote DRAM Accesses
   0   0.35        676 M     1950 M       762 K              1127 K
   1   0.17       1860 M       10 G      1623 K              5550 K
   2   0.33        572 M     1746 M      1027 K               734 K
   3   0.32        425 M     1343 M       646 K               666 K
   4   0.23       1814 K     7827 K      4269                2080
   5   0.17       1823 M       10 G      1647 K              5607 K
   6   0.29         14 M       49 M        24 K                20 K
   7   0.32        124 M      393 M       187 K               195 K
   8   0.22       2037 K     9146 K      4756                2185
   9   0.17       1844 M       10 G      1687 K              5515 K
  10   0.09        477 K     5377 K      2156                 934
  11   0.36       6417 K       17 M        13 K              4969
  12   0.11        631 K     5680 K      2870                1350
  13   0.18       1955 M       11 G      1810 K              5900 K
  14   0.07       1216 K       17 M      2295                2341
  15   0.11       8875 K       77 M      8590                3438
  16   0.08        367 K     4856 K      2068                 780
  17   0.17       1839 M       10 G      1735 K              5637 K
  18   0.17       1160 K     6757 K      2765                1180
  19   0.32       4925 K       15 M        11 K              3897
  20   0.14        854 K     6239 K      2794                1573
  21   0.17       1892 M       10 G      1806 K              5720 K
  22   0.33        602 M     1818 M       940 K               847 K
  23   0.31       5893 K       19 M        14 K              5000
  24   0.18       1144 K     6518 K      3125                1257
  25   0.17       1882 M       10 G      1853 K              5742 K
  26   0.22       3117 K       14 M        12 K              3816
  27   0.31       8291 K       26 M        20 K              8154
  28   0.25       2021 K     8122 K      3989                2336
  29   0.18       1938 M       10 G      1827 K              5724 K
  30   0.29       6271 K       21 M        13 K              5992
  31   0.21       6604 K       31 M        13 K              5231
  32   0.01        568 K       77 M       474                 335
  33   0.18       1928 M       10 G      1675 K              5688 K
  34   0.00        351 K       96 M      1679                 377
  35   0.01        544 K       62 M      2896                 605
  36   0.25       2803 K       11 M      5209                2069
  37   0.18       1976 M       11 G      1810 K              6000 K
  38   0.13       1090 K     8538 K      3891                1188
  39   0.07       2211 K       29 M      8884                2924
  40   0.20       1486 K     7391 K      3894                1366
  41   0.18       2000 M       11 G      1800 K              5910 K
  42   0.12        614 K     5311 K      2663                1068
  43   0.07        351 K     5041 K      4732                1085
  44   0.12        575 K     4930 K      1963                 741
  45   0.17       1851 M       10 G      1729 K              5547 K
  46   0.03        393 K       12 M      1760                 742
  47   0.01        497 K       47 M      3039                 949
  48   0.16        899 K     5609 K      2809                1037
  49   0.18       1961 M       11 G      1850 K              5908 K
  50   0.08        402 K     4840 K      2096                 700
  51   0.07        353 K     4943 K      4166                 770
  52   0.11        575 K     5100 K      2016                 628
  53   0.17       1936 M       11 G      1841 K              5760 K
  54   0.00        435 K      102 M       398                 444
  55   0.07        398 K     5331 K      5844                 945
  56   0.10        474 K     4794 K      2208                 612
  57   0.16       1688 M       10 G      1616 K              5241 K
  58   0.07        391 K     5441 K      6096                1005
  59   0.07        417 K     5742 K      7805                1292
  60   0.14        743 K     5432 K      2505                1074
  61   0.18       1915 M       10 G      1813 K              5709 K
  62   0.09        609 K     6668 K      6467                1322
  63   0.04        579 K       15 M      4991                1207
-------------------------------------------------------------------------------------------------------------------
   *   0.18         32 G      182 G        31 M                94 M

^CDEBUG: caught signal to interrupt (Interrupt).
Cleaning up
 Zeroed uncore PMU registers
 Re-enabling NMI watchdog.
[root@R820-04 yum.repos.d]#
``` 
    
    
    Remote DRAM Accesses 和 Local DRAM Accesses 的比值上升差异很大. 

但IPC差异不大 (0.20-> 0.18), 不足以作为判断标准 (perf结果也显示IPC差异不大)

通过perf监控NUMA的命中次数: 

```
[root@R820-04 ~]# perf stat -e node-load-misses,node-loads,node-prefetch-misses,node-prefetches,node-store-misses,node-stores -p $(pgrep mysqld$) -- sleep 5

 Performance counter stats for process id '60449':

        29,558,555      node-load-misses                                              (20.76%)
        36,067,534      node-loads                                                    (20.77%)
         3,265,073      node-prefetch-misses                                          (20.76%)
         3,389,198      node-prefetches                                               (20.85%)
        11,858,589      node-store-misses                                             (20.90%)
        23,094,490      node-stores                                                   (20.82%)

       5.006295895 seconds time elapsed
``` 
    
    
    node-load-misses 率升高 (29,558,555 / 36,067,534 = 82%), 之前是 (8,684,148 / 37,547,906 = 23%)

# 测试 innodb-numa-interleave

将 innodb-numa-interleave 设置为 1

将cpu绑定在node1上

```
taskset -apc 1,5,9,13,17,21,25,29,33,37,41,45,49,53,57,61 $(pgrep mysqld$) #node 1
``` 

并进行sysbench压力

```
sysbench /usr/share/sysbench/oltp_point_select.lua --mysql-host=127.0.0.1 --mysql-port=8022 --mysql-user=msandbox --mysql-password=msandbox --mysql-db=test --tables=64 --table-size=1000000 --report-interval=3 --threads=16 --time=300 run
``` 

观察numastat: 

![image2022-7-6 10:35:20.png](/assets/01KJBYJTJXB03VRDH5EHQ3VQWY/image2022-7-6%2010%3A35%3A20.png)

buffer pool会均匀分布于各个node中

# 测试 numa_balancing

将 /proc/sys/kernel/numa_balancing 设置为 1

将cpu绑定在node1上

```
taskset -apc 1,5,9,13,17,21,25,29,33,37,41,45,49,53,57,61 $(pgrep mysqld$) #node 1
``` 

并进行sysbench压力

```
sysbench /usr/share/sysbench/oltp_point_select.lua --mysql-host=127.0.0.1 --mysql-port=8022 --mysql-user=msandbox --mysql-password=msandbox --mysql-db=test --tables=64 --table-size=1000000 --report-interval=3 --threads=16 --time=300 run
``` 

查看numastat: 

![image2022-7-6 14:34:11.png](/assets/01KJBYJTJXB03VRDH5EHQ3VQWY/image2022-7-6%2014%3A34%3A11.png)

将cpu换绑在node3上

```
taskset -apc 3,7,11,15,19,23,27,31,35,39,43,47,51,55,59,63 $(pgrep mysqld$) #node 3
``` 

持续进行sysbench压力, 一定时间后, 可以看到numastat的变化: 

![image2022-7-6 14:35:1.png](/assets/01KJBYJTJXB03VRDH5EHQ3VQWY/image2022-7-6%2014%3A35%3A1.png)

numa分布发生了转移

转移速度大概1M/s

对比: numa_balancing=0时, node1中的数据不会发生转移

# perf 输出学习

  - <https://andreask.cs.illinois.edu/Teaching/HPCFall2012/lec10.pdf>
    - stalled-cycles-frontend: cycles Instruction fetch stalls. Should never happen–means CPU could not predict where code is going. (→ pipeline stall)
      - 获取指令时阻塞
    - stalled-cycles-backend: cycles Execution units (ALU/FPU/Load-store) is waiting for data/computation/. . .
      - 获取数据时阻塞
    - CPU处理流程: 
      - ![image2022-6-22 15:29:12.png](/assets/01KJBYJTJXB03VRDH5EHQ3VQWY/image2022-6-22%2015%3A29%3A12.png)
    - <https://www.intel.com/content/www/us/en/develop/documentation/vtune-cookbook/top/methodologies/top-down-microarchitecture-analysis-method.html>
    - ![image](https://www.intel.com/content/dam/dita/develop/vtune-cookbook-0601-1635/050E7764-A92D-41AC-B32F-09762FBD11EC.png/_jcr_content/renditions/original)
