---
title: 20220528 - page_cleaner日志的模拟方法
confluence_page_id: 1737184
created_at: 2022-05-28T08:16:28+00:00
updated_at: 2022-05-28T11:13:44+00:00
---

研究 page_cleaner 日志连续打印的周期

日志样例: 

```
2022-05-28T07:53:24.667318Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5025ms. The settings might not be optimal. (flushed=0 and evicted=0, during the time.)
2022-05-28T07:53:49.357708Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5017ms. The settings might not be optimal. (flushed=0 and evicted=0, during the time.)
2022-05-28T07:54:34.532774Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5025ms. The settings might not be optimal. (flushed=0 and evicted=0, during the time.)
2022-05-28T07:55:59.873450Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5018ms. The settings might not be optimal. (flushed=0 and evicted=0, during the time.)
2022-05-28T07:58:45.648888Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5029ms. The settings might not be optimal. (flushed=12 and evicted=0, during the time.)
``` 

代码逻辑: (版本 5.7.26)

![image2022-5-28 16:10:15.png](/assets/01KJBYJEQN14KSK24XPHFC6S1M/image2022-5-28%2016%3A10%3A15.png)

使用gdb调试 mysqld-debug 版本: 

在进入刷盘计算周期时, 等待5s

```
break buf0flu.cc:3188
commands
shell sleep 5
c
end
``` 

使得每个计算周期都触发以下条件: 

  - ret_sleep == OS_SYNC_TIME_EXCEEDED
  - curr_time > next_loop_time + 3000

打印内部计算周期信息: 

```
break buf0flu.cc:3210
commands
p curr_time - next_loop_time
p warn_count
p warn_interval
c
end
``` 

周期信息示例: 

```
Thread 13 "mysqld" hit Breakpoint 2, buf_flush_page_cleaner_coordinator (arg=<optimized out>) at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/buf/buf0flu.cc:3210
3210	in /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/buf/buf0flu.cc
$649 = 4017
$650 = 45
$651 = 128

---

Thread 13 "mysqld" hit Breakpoint 2, buf_flush_page_cleaner_coordinator (arg=<optimized out>) at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/buf/buf0flu.cc:3210
3210	in /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/buf/buf0flu.cc
$652 = 4016
$653 = 44
$654 = 128
``` 

结论: 

```
2022-05-28T07:53:06.151617Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 53466ms. The settings might not be optimal. (flushed=100 and evicted=0, during the time.)
2022-05-28T07:53:24.667318Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5025ms. The settings might not be optimal. (flushed=0 and evicted=0, during the time.)
2022-05-28T07:53:49.357708Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5017ms. The settings might not be optimal. (flushed=0 and evicted=0, during the time.)
2022-05-28T07:54:34.532774Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5025ms. The settings might not be optimal. (flushed=0 and evicted=0, during the time.)
2022-05-28T07:55:59.873450Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5018ms. The settings might not be optimal. (flushed=0 and evicted=0, during the time.)
2022-05-28T07:58:45.648888Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5029ms. The settings might not be optimal. (flushed=12 and evicted=0, during the time.)
2022-05-28T08:04:12.269993Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5021ms. The settings might not be optimal. (flushed=4 and evicted=0, during the time.)
2022-05-28T08:14:59.773697Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5016ms. The settings might not be optimal. (flushed=0 and evicted=0, during the time.)
2022-05-28T08:36:29.478936Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5018ms. The settings might not be optimal. (flushed=0 and evicted=0, during the time.)
2022-05-28T09:19:24.291531Z 0 [Note] InnoDB: page_cleaner: 1000ms intended loop took 5034ms. The settings might not be optimal. (flushed=36 and evicted=22, during the time.)
```
