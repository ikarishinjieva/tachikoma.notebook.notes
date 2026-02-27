---
title: 20220818 - 百胜slave延迟, 关于上下文切换的诊断
confluence_page_id: 1933662
created_at: 2022-08-18T07:52:49+00:00
updated_at: 2022-08-18T11:06:33+00:00
---

在测试环境进行试验 (10.186.17.125, CPU是skylake架构)

# 现象

架构: 1主1从. 使用 程序 ([20220811 - golang产生对MySQL的并发连接代码]) 向slave建立40000根空闲连接 (空闲连接每秒发起一次COM_PING), 使用sysbench向master发起压力: 

```
sysbench /usr/share/sysbench/oltp_update_index.lua --mysql-host=127.0.0.1 --mysql-port=18822 --mysql-user=msandbox --mysql-password=msandbox --mysql-db=test --table-size=1000000 --tables=200 --threads=200 --db-ps-mode=disable --auto_inc=off --report-interval=3 --max-requests=0 --time=1200 --percentile=95 --skip_trx=off --mysql-ignore-errors=6002,6004,4012,2013,4016,1062,1213 --create_secondary=off run
``` 

发现master的性能在下降

**建立空闲连接前:**

sysbench压力: 

```
[ 3s ] thds: 200 tps: 35095.97 qps: 35095.97 (r/w/o: 0.00/35095.97/0.00) lat (ms,95%): 8.58 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 200 tps: 33235.18 qps: 33235.18 (r/w/o: 0.00/33235.18/0.00) lat (ms,95%): 10.27 err/s: 0.00 reconn/s: 0.00
[ 9s ] thds: 200 tps: 33106.49 qps: 33106.49 (r/w/o: 0.00/33106.49/0.00) lat (ms,95%): 12.08 err/s: 0.00 reconn/s: 0.00
[ 12s ] thds: 200 tps: 26590.09 qps: 26590.09 (r/w/o: 0.00/26590.09/0.00) lat (ms,95%): 11.45 err/s: 0.00 reconn/s: 0.00
``` 

系统级perf:

```
root@R740-25:~/sandboxes/rsandbox_5_7_21# perf stat -a -- sleep 5

 Performance counter stats for 'system wide':

     400200.617922      cpu-clock (msec)          #   79.939 CPUs utilized
         5,229,482      context-switches          #    0.013 M/sec
           774,506      cpu-migrations            #    0.002 M/sec
           661,805      page-faults               #    0.002 M/sec
   490,406,866,208      cycles                    #    1.225 GHz
   209,097,911,497      instructions              #    0.43  insn per cycle
    41,685,796,695      branches                  #  104.162 M/sec
     1,097,097,147      branch-misses             #    2.63% of all branches

       5.006336734 seconds time elapsed
``` 

master的perf:

```
root@R740-25:~/sandboxes/rsandbox_5_7_21# perf stat -p $(cat /data/huangyan/sandboxes/rsandbox_5_7_21/master/data/mysql_sandbox18822.pid) -- sleep 5

 Performance counter stats for process id '1936049':

      44832.119064      task-clock (msec)         #    8.954 CPUs utilized
           929,488      context-switches          #    0.021 M/sec
           345,585      cpu-migrations            #    0.008 M/sec
            32,948      page-faults               #    0.735 K/sec
   135,127,198,876      cycles                    #    3.014 GHz
    35,742,947,216      instructions              #    0.26  insn per cycle
     6,428,242,433      branches                  #  143.385 M/sec
       258,039,625      branch-misses             #    4.01% of all branches

       5.006772173 seconds time elapsed
``` 

slave的perf: 

```
root@R740-25:~/sandboxes/rsandbox_5_7_21# perf stat -p $(cat /data/huangyan/sandboxes/rsandbox_5_7_21/node1/data/mysql_sandbox18823.pid) -- sleep 5

 Performance counter stats for process id '1945642':

      40613.216408      task-clock (msec)         #    8.114 CPUs utilized
         1,029,998      context-switches          #    0.025 M/sec
           142,975      cpu-migrations            #    0.004 M/sec
            15,053      page-faults               #    0.371 K/sec
   122,284,032,711      cycles                    #    3.011 GHz
    52,168,846,218      instructions              #    0.43  insn per cycle
     9,966,639,929      branches                  #  245.404 M/sec
       205,784,246      branch-misses             #    2.06% of all branches

       5.005256508 seconds time elapsed
``` 

**建立空闲连接后:**

sysbench压力: 

```
[ 510s ] thds: 200 tps: 13740.71 qps: 13740.71 (r/w/o: 0.00/13740.71/0.00) lat (ms,95%): 39.65 err/s: 0.00 reconn/s: 0.00
[ 513s ] thds: 200 tps: 14156.64 qps: 14156.64 (r/w/o: 0.00/14156.64/0.00) lat (ms,95%): 41.85 err/s: 0.00 reconn/s: 0.00
[ 516s ] thds: 200 tps: 8971.06 qps: 8971.06 (r/w/o: 0.00/8971.06/0.00) lat (ms,95%): 40.37 err/s: 0.00 reconn/s: 0.00
[ 519s ] thds: 200 tps: 16036.29 qps: 16036.29 (r/w/o: 0.00/16036.29/0.00) lat (ms,95%): 32.53 err/s: 0.00 reconn/s: 0.00
``` 

系统级perf:

```
root@R740-25:~/sandboxes/rsandbox_5_7_21# perf stat -a -- sleep 5

 Performance counter stats for 'system wide':

     399996.192657      cpu-clock (msec)          #   79.931 CPUs utilized
         2,211,632      context-switches          #    0.006 M/sec
           336,914      cpu-migrations            #    0.842 K/sec
           272,914      page-faults               #    0.682 K/sec
 1,124,039,335,437      cycles                    #    2.810 GHz
   358,706,211,197      instructions              #    0.32  insn per cycle
    66,682,953,524      branches                  #  166.709 M/sec
     1,675,093,710      branch-misses             #    2.51% of all branches

       5.004290794 seconds time elapsed
``` 

  - context-switches / cpu-migrations / page_faults 都在下降
  - cycles / instructions / branches上升
  - IPC下降  
  

master的perf:

```
root@R740-25:~/sandboxes/rsandbox_5_7_21# perf stat -p $(cat /data/huangyan/sandboxes/rsandbox_5_7_21/master/data/mysql_sandbox18822.pid) -- sleep 5

 Performance counter stats for process id '1936049':

      31035.866729      task-clock (msec)         #    6.198 CPUs utilized
           572,428      context-switches          #    0.018 M/sec
           171,542      cpu-migrations            #    0.006 M/sec
            13,930      page-faults               #    0.449 K/sec
    93,589,609,733      cycles                    #    3.016 GHz
    24,748,693,916      instructions              #    0.26  insn per cycle
     4,452,935,112      branches                  #  143.477 M/sec
       193,934,961      branch-misses             #    4.36% of all branches

       5.007075051 seconds time elapsed
``` 

  - task-clock下降, CPU利用率变低
  - page-faults 速率 变低, 其他项 速率变化不大, 速率变低
  - IPC没变化

slave的perf: 

```
root@R740-25:~/sandboxes/rsandbox_5_7_21# perf stat -p $(cat /data/huangyan/sandboxes/rsandbox_5_7_21/node1/data/mysql_sandbox18823.pid) -- sleep 5

 Performance counter stats for process id '1945642':

      32348.054696      task-clock (msec)         #    5.140 CPUs utilized
           704,171      context-switches          #    0.022 M/sec
           192,182      cpu-migrations            #    0.006 M/sec
            24,731      page-faults               #    0.765 K/sec
    94,875,178,065      cycles                    #    2.933 GHz
    35,830,116,443      instructions              #    0.38  insn per cycle
     6,867,364,649      branches                  #  212.296 M/sec
       167,502,612      branch-misses             #    2.44% of all branches

       6.293324700 seconds time elapsed
``` 

  - task-clock下降, CPU利用率变低
  - page-faults 速率 变高, 其他项 速率变化不大, 速率变低
  - perf采集时间变长

**当把COM_PING去除**

对性能影响变轻

# 结论

发现 制造空闲连接的工具, 会在同一台机器造成过大CPU压力, 导致一系列现象: 

  - CPU sys占 50%+
  - 整体 cycles 升高, 使用了更多CPU
  - 整体 context-switches / cpu-migrations / page_faults 等降低, 更多的陷入了syscall中
  - 各进程的 task-clock / cycles 降低, 使用了更少的CPU
  - 整体 IPC 下降的原因不明

# 其他信息

  - 检查全局线程数: 

```
ps -elfT | wc -l
```
