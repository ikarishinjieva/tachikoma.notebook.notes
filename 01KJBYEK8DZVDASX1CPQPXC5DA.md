---
title: 20210830 - THP的作用探索
confluence_page_id: 1343692
created_at: 2021-08-29T16:48:36+00:00
updated_at: 2021-09-08T15:16:12+00:00
---

# 如何观察THP对TLB的作用 - c代码: 

代码: 

```
#include <stdlib.h>

#define TWOMB 2097152

int main () {
    srand(1000);
    int size = TWOMB * 1000;
    int *arr = (int*)malloc(size);
    for (int i = 0; i < 10000000; i++) {
        int offset = rand() % (size / sizeof(int));
        arr[offset] = rand();
    }

    return 0;
}
``` 

测试命令: 

```
[root@R820-04 tmp]# echo never > /sys/kernel/mm/transparent_hugepage/enabled
[root@R820-04 tmp]# perf stat -e cycles,instructions,dtlb_load_misses.walk_duration,dTLB-loads,dTLB-load-misses,dtlb_store_misses.walk_duration,dTLB-stores,dTLB-store-misses -- ./a.out

 Performance counter stats for './a.out':

     6,308,570,056      cycles                                                        (37.47%)
     2,923,013,202      instructions              #    0.46  insn per cycle                                              (49.99%)
       285,998,437      dtlb_load_misses.walk_duration                                     (50.02%)
       842,429,888      dTLB-loads                                                    (50.03%)
        14,802,132      dTLB-load-misses          #    1.76% of all dTLB cache hits   (25.07%)
     1,043,912,486      dtlb_store_misses.walk_duration                                     (25.05%)
       646,535,646      dTLB-stores                                                   (24.98%)
        12,526,698      dTLB-store-misses                                             (24.93%)

       2.454284884 seconds time elapsed
``` 
    
    
    dtlb_load_misses.walk_duration/cycles = 4.5%
    
    
    dtlb_store_misses.walk_duration/cycles = 16.5%

TLB miss引起的page walk, 成本为整体任务的21%

```
[root@R820-04 tmp]# echo always > /sys/kernel/mm/transparent_hugepage/enabled
[root@R820-04 tmp]# perf stat -e cycles,instructions,dtlb_load_misses.walk_duration,dTLB-loads,dTLB-load-misses,dtlb_store_misses.walk_duration,dTLB-stores,dTLB-store-misses -- ./a.out

 Performance counter stats for './a.out':

     4,076,911,394      cycles                                                        (37.44%)
     1,212,014,374      instructions              #    0.30  insn per cycle                                              (49.96%)
         9,559,625      dtlb_load_misses.walk_duration                                     (50.09%)
       383,245,445      dTLB-loads                                                    (50.08%)
            56,432      dTLB-load-misses          #    0.01% of all dTLB cache hits   (25.02%)
       131,354,071      dtlb_store_misses.walk_duration                                     (25.02%)
       352,949,774      dTLB-stores                                                   (24.96%)
         9,810,053      dTLB-store-misses                                             (24.99%)

       1.652983652 seconds time elapsed
``` 
    
    
    dtlb_load_misses.walk_duration/cycles = 0.2%
    
    
    dtlb_store_misses.walk_duration/cycles = 3.2%

有明显效果

# 如何观察THP对TLB的作用 - sysbench

TLB的大小: 通过cpuid查看

```
   cache and TLB information (2):
      0x5a: data TLB: 2M/4M pages, 4-way, 32 entries
      0x03: data TLB: 4K pages, 4-way, 64 entries
      0x76: instruction TLB: 2M/4M pages, fully, 8 entries
      0xff: cache data is in CPUID 4
      0xb2: instruction TLB: 4K, 4-way, 64 entries
      0xf0: 64 byte prefetching
      0xca: L2 TLB: 4K pages, 4-way, 512 entries
``` 

可以看到TLB分为: data TLB(dTLB), instruction TLB(iTLB), L2 TLB. 使用了两级TLB

对THP作用进行测试: 

测试命令: 

```
[root@R820-04 tmp]# echo never > /sys/kernel/mm/transparent_hugepage/enabled
[root@R820-04 tmp]# perf stat -e cycles,instructions,dtlb_load_misses.walk_duration,dTLB-loads,dTLB-load-misses,dtlb_store_misses.walk_duration,dTLB-stores,dTLB-store-misses -- sysbench memory --memory-block-size=4M --memory-total-size=1000G --time=500 --threads=32 --memory-oper=read run
sysbench 1.0.20 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 32
Initializing random number generator from current time

Running memory speed test with the following options:
  block size: 4096KiB
  total size: 1024000MiB
  operation: read
  scope: global

Initializing worker threads...

Threads started!

Total operations: 256000 (65226.77 per second)

1024000.00 MiB transferred (260907.09 MiB/sec)

General statistics:
    total time:                          3.9191s
    total number of events:              256000

Latency (ms):
         min:                                    0.27
         avg:                                    0.44
         max:                                   38.92
         95th percentile:                        0.64
         sum:                               112812.68

Threads fairness:
    events (avg/stddev):           8000.0000/0.00
    execution time (avg/stddev):   3.5254/0.10

 Performance counter stats for 'sysbench memory --memory-block-size=4M --memory-total-size=1000G --time=500 --threads=32 --memory-oper=read run':

   257,301,612,673      cycles                                                        (34.75%)
   185,687,473,082      instructions              #    0.72  insn per cycle                                              (47.25%)
     4,848,078,695      dtlb_load_misses.walk_duration                                     (49.77%)
   135,010,536,882      dTLB-loads                                                    (48.38%)
       276,791,856      dTLB-load-misses          #    0.21% of all dTLB cache hits   (33.15%)
        12,321,096      dtlb_store_misses.walk_duration                                     (28.25%)
       113,072,475      dTLB-stores                                                   (28.58%)
           610,363      dTLB-store-misses                                             (27.80%)

       3.942367419 seconds time elapsed

[root@R820-04 tmp]#
```
```
[root@R820-04 tmp]# echo always > /sys/kernel/mm/transparent_hugepage/enabled
[root@R820-04 tmp]# perf stat -e cycles,instructions,dtlb_load_misses.walk_duration,dTLB-loads,dTLB-load-misses,dtlb_store_misses.walk_duration,dTLB-stores,dTLB-store-misses -- sysbench memory --memory-block-size=4M --memory-total-size=1000G --time=500 --threads=32 --memory-oper=read run
sysbench 1.0.20 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 32
Initializing random number generator from current time

Running memory speed test with the following options:
  block size: 4096KiB
  total size: 1024000MiB
  operation: read
  scope: global

Initializing worker threads...

Threads started!

Total operations: 256000 (73954.90 per second)

1024000.00 MiB transferred (295819.59 MiB/sec)

General statistics:
    total time:                          3.4562s
    total number of events:              256000

Latency (ms):
         min:                                    0.27
         avg:                                    0.39
         max:                                   14.74
         95th percentile:                        0.59
         sum:                                99874.01

Threads fairness:
    events (avg/stddev):           8000.0000/0.00
    execution time (avg/stddev):   3.1211/0.13

 Performance counter stats for 'sysbench memory --memory-block-size=4M --memory-total-size=1000G --time=500 --threads=32 --memory-oper=read run':

   229,003,407,724      cycles                                                        (35.99%)
   185,747,455,560      instructions              #    0.81  insn per cycle                                              (48.49%)
     1,230,312,136      dtlb_load_misses.walk_duration                                     (50.79%)
   134,941,904,905      dTLB-loads                                                    (47.46%)
        62,290,684      dTLB-load-misses          #    0.05% of all dTLB cache hits   (30.66%)
         9,470,501      dtlb_store_misses.walk_duration                                     (27.52%)
        97,046,166      dTLB-stores                                                   (27.51%)
           393,133      dTLB-store-misses                                             (26.70%)

       3.477416344 seconds time elapsed

[root@R820-04 tmp]#
``` 

# 如何观察THP对TLB的作用2

使用perf对MySQL进行观察, 重点是PMC: dtlb_load_misses.walk_duration

```
perf stat -e cycles,instructions,dtlb_load_misses.walk_duration,dTLB-loads,dTLB-load-misses,dtlb_store_misses.walk_duration,dTLB-stores,dTLB-store-misses -p `pidof mysqld` -- sleep 5
``` 

使用sysbench: 

```
sysbench /usr/share/sysbench/oltp_point_select.lua --mysql-host=127.0.0.1 --mysql-port=8022 --mysql-user=msandbox --mysql-password=msandbox --mysql-db=sbtest2 --tables=500 --table-size=500000 --threads=50 --db-ps-mode=disable --auto_inc=off --report-interval=3 --max-requests=0 --time=50 --percentile=95 --skip_trx=off --mysql-ignore-errors=6002,6004,4012,2013,4016,1062 --create_secondary=off run
``` 

当/sys/kernel/mm/transparent_hugepage/enabled策略为never: 

等待buffer pool填充到20G, 查看数值:

  1. sysbench结果: QPS: 54706, 不稳定
  2. perf结果: 

```
[root@R820-04 msb_8_0_22]# perf stat -e cycles,instructions,dtlb_load_misses.walk_duration,dTLB-loads,dTLB-load-misses,dtlb_store_misses.walk_duration,dTLB-stores,dTLB-store-misses -p `pidof mysqld` -- sleep 5

 Performance counter stats for process id '48058':

   141,222,909,484      cycles                                                        (32.31%)
    36,654,834,455      instructions              #    0.26  insn per cycle                                              (44.85%)
    10,437,153,696      dtlb_load_misses.walk_duration                                     (48.96%)
     9,550,965,537      dTLB-loads                                                    (43.66%)
       227,081,161      dTLB-load-misses          #    2.38% of all dTLB cache hits   (32.14%)
     1,019,677,317      dtlb_store_misses.walk_duration                                     (29.36%)
     7,087,081,923      dTLB-stores                                                   (29.42%)
        26,393,696      dTLB-store-misses                                             (28.21%)

       5.005397787 seconds time elapsed
``` 

TLB miss引起的代价为: dtlb_load_misses.walk_duration/cycle = 7%

当/sys/kernel/mm/transparent_hugepage/enabled策略为always: 

等待buffer pool填充到20G, 查看数值:

  1. sysbench结果: QPS: 56045, 不稳定
  2. perf结果: 

```
[root@R820-04 msb_8_0_22]# perf stat -e cycles,instructions,dtlb_load_misses.walk_duration,dTLB-loads,dTLB-load-misses,dtlb_store_misses.walk_duration,dTLB-stores,dTLB-store-misses -p `pidof mysqld` -- sleep 5

 Performance counter stats for process id '48695':

   138,911,912,791      cycles                                                        (32.42%)
    36,872,965,737      instructions              #    0.27  insn per cycle                                              (45.00%)
     7,911,367,313      dtlb_load_misses.walk_duration                                     (48.93%)
     9,584,316,127      dTLB-loads                                                    (43.71%)
       187,438,222      dTLB-load-misses          #    1.96% of all dTLB cache hits   (32.27%)
       682,320,625      dtlb_store_misses.walk_duration                                     (29.27%)
     7,154,163,864      dTLB-stores                                                   (29.30%)
        18,473,578      dTLB-store-misses                                             (28.17%)

       5.006047270 seconds time elapsed
``` 

TLB miss引起的代价为: dtlb_load_misses.walk_duration/cycle = 5%

验证: THP大小: 

```
cat /proc/`pidof mysqld`/smaps | grep AnonHugePages | awk '{s+=$2} END {print s}'
``` 

# 暂时结论

使用c程序, 可以看到性能有明显差异

使用MySQL, 看不到THP对性能的明显影响, 猜测有以下原因: 

  1. 测试场景不能 产生大量的 TLB miss (从MySQL实验的数值上看, TLB miss率已经足够高, 与c场景类似)
  2. 测试场景中, THP带来的TLB miss提升不大 (c场景中, 提升为几百倍, 而MySQL场景中, 提升为20%)
  3. THP带来的性能提升, 对于整体SQL的处理效率, 数量级太小 (dtlb_load_misses.walk_duration/cycle 为 7%左右, 即使完全消除, 性能提升也小于10%)
