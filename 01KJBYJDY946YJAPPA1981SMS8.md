---
title: 20220508 - 杭州银行 MySQL slave 崩溃
confluence_page_id: 1737172
created_at: 2022-05-08T06:19:59+00:00
updated_at: 2022-05-08T16:39:24+00:00
---

# 错误日志

[附件: 43712d66ac57775b23bded6e2699f716_158512e1086aad9f3aaf3efc54436685_8.err.zip] 

# 分析

大量线程阻塞在 LOCK_SYS_WAIT

```
2022-05-05T22:53:56.281380+08:00 0 [Warning] [MY-012985] [InnoDB] A long semaphore wait:
--Thread 140005531973376 has waited at lock0wait.cc line 74 for 923 seconds the semaphore:
Mutex at 0x7f55bb54d0d8, Mutex LOCK_SYS_WAIT created lock0lock.cc:331, lock var 1
``` 

但报错是: "Semaphore wait has lasted > 600 seconds. We intentionally crash the server because it appears to be hung"

解释: "A long semaphore wait" 每30s打印一次, 打印10次后, 触发panic. 也就是900 - 930秒, 才会产生崩溃

\-----

对lock_wait_mutex_enter的调用进行分析

```
lock_wait_mutex_enter()
	- lock_wait_table_release_slot: 没有耗时操作
		- lock_wait_suspend_thread
			- ...

	- lock_wait_suspend_thread: 不涉及 其他两把锁
		- ...

	- lock_wait_check_slots_for_timeouts: 涉及Global_exclusive_latch_guard, 可能有耗时操作
		- lock_wait_timeout_thread
			- srv0start

	- lock_wait_snapshot_waiting_threads: 没有耗时操作
		- lock_wait_update_schedule_and_check_for_deadlocks
			- lock_wait_timeout_thread
				- srv0start

	- lock_wait_publish_new_weights: 没有耗时操作
		- lock_wait_compute_and_publish_weights_except_cycles
			- lock_wait_update_schedule_and_check_for_deadlocks
				- lock_wait_timeout_thread
					- srv0start

	- lock_wait_check_candidate_cycle: 没有耗时操作
		- lock_wait_find_and_handle_deadlocks
			- lock_wait_update_schedule_and_check_for_deadlocks
				- lock_wait_timeout_thread
					- srv0start
``` 

\------

可能与 lock_wait_timeout_thread 卡顿有关, 没有有效指标, 判断lock_wait_timeout_threads是否卡顿

\------

在日志中, 涉及三类锁: 

  - 锁时间短, 且基本同频
    - TRX_SYS: trx_sys->mutex
      - mutex_enter(&trx_sys->mutex)
        - lock_rec_other_trx_holds_expl : 只有debug模式有效
        - ...
      - trx_sys_mutex_enter
        - p_s在扫lock ??
    - locksys::Global_exclusive_latch_guard (sync0sharded_rw.h line 120) (推测)
  - LOCK_SYS_WAIT: 锁时间最长

TODO: 找到代码路径, 在 LOCK_SYS_WAIT 保护下, 同时获取另外两把锁

\------

日志里有clone0repl的调用??

gtid persister 放在了clone模块, 原因不详... 

\------

文件中有207个 "INNODB MONITOR OUTPUT", 也就是说 等待有几轮不会超过900s, 直到最后一轮超过900s后崩溃

推论: 锁状态是可以自行恢复的

切分日志: 

```
csplit -n 3 -k  43712d66ac57775b23bded6e2699f716_158512e1086aad9f3aaf3efc54436685_8.err '/00 INNODB MONITOR OUTPUT/' '{500}'
``` 

找到最大的文件: xx163

查看有SQL显示的线程 140005063235328: 

```
---TRANSACTION 4698729322, ACTIVE 1175 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 1136, 1 row lock(s)
MySQL thread id 2165595, OS thread handle 140005063235328, query id 328153169 168.68.14.23 bapp update
insert into EEL_IMAS_SEND_MSG
     ( SEND_ORGANIZATION,

        SEND_IDENTIFICATION,

        RECV_ORGANIZATION,

        RECV_IDENTIFICATION,
``` 

相关的锁信息: 

```
--Thread 140005063235328 has waited at lock0wait.cc line 215 for 595 seconds the semaphore:
Mutex at 0x7f55bb54d0d8, Mutex LOCK_SYS_WAIT created lock0lock.cc:331, lock var 1
``` 

这个LOCK_SYS_WAIT的等待, 是 lock_wait_table_release_slot 或者 lock_wait_suspend_thread 触发的等待, 但不能查明持有方
