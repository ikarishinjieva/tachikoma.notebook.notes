---
title: 20210329 - MySQL刷盘研究 - 数据文件刷盘慢导致SQL卡顿
confluence_page_id: 753674
created_at: 2021-03-29T04:26:21+00:00
updated_at: 2021-04-04T07:20:35+00:00
---

# 参考文档

  - <https://www.percona.com/blog/2020/01/22/innodb-flushing-in-action-for-percona-server-for-mysql/>

# 分析

  - innodb_data_pending_fsyncs = fil_n_pending_log_flushes + fil_n_pending_tablespace_flushes
    - fil_n_pending_log_flushes: 与刷盘类型 FIL_TYPE_LOG 相关, 等于 innodb_os_log_pending_fsyncs
      - checkpoint_now_set
      - logs_empty_and_mark_files_at_shutdown
      - create_log_files
      - innobase_start_or_create_for_mysql
    - fil_n_pending_tablespace_flushes: 与刷盘类型 FIL_TYPE_IMPORT/FIL_TYPE_TABLESPACE 相关
    - innodb engine status的输出

```
"Pending flushes (fsync) log: " ULINTPF "; "
		"buffer pool: " ULINTPF "\n"
		ULINTPF " OS file reads, "
		ULINTPF " OS file writes, "
		ULINTPF " OS fsyncs\n",
		fil_n_pending_log_flushes,
		fil_n_pending_tablespace_flushes,
		os_n_file_reads,
		os_n_file_writes,
		os_n_fsyncs);
```

# 将数据文件的flush延迟增到1s, 会发生SQL卡顿

获取MySQL卡顿时的堆栈: 

```
Thread 37 (Thread 0x7fa6e1faa700 (LWP 9840)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>,
    mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>)
    at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc",
    src_line=152, that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection ()
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6b993a0)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b80910)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6e1faa700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 36 (Thread 0x7fa6c1188700 (LWP 9833)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x6a0ab48) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x6a0aaf8, cond=0x6a0ab20) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x6a0ab20, mutex=mutex@entry=0x6a0aaf8)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x6a0aae8)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x6a0aae8, reset_sig_count=reset_sig_count@entry=10)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=reset_sig_count@entry=10)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x000000000116c954 in fil_flush (space_id=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:6025
---Type <return> to continue, or q <return> to quit---
#7  0x0000000000f81c82 in log_write_flush_to_disk_low ()
    at /usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc:1164
#8  log_write_up_to (lsn=<optimized out>, flush_to_disk=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc:1414
#9  0x00000000010a1e92 in trx_flush_log_if_needed_low (lsn=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/trx/trx0trx.cc:1803
#10 trx_flush_log_if_needed (trx=0x7fa76bbff750, lsn=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/trx/trx0trx.cc:1825
#11 trx_commit_complete_for_mysql (trx=trx@entry=0x7fa76bbff750)
    at /usr/src/mysql-5.7.32/storage/innobase/trx/trx0trx.cc:2460
#12 0x0000000000f30393 in innobase_commit (hton=<optimized out>, thd=0x7fa67c018bc0,
    commit_trx=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/handler/ha_innodb.cc:4458
#13 0x00000000007d1174 in ha_commit_low (thd=0x7fa67c018bc0, all=<optimized out>,
    run_after_commit=run_after_commit@entry=true) at /usr/src/mysql-5.7.32/sql/handler.cc:1911
#14 0x0000000000cf2e04 in TC_LOG_DUMMY::commit (this=<optimized out>, thd=<optimized out>,
    all=<optimized out>) at /usr/src/mysql-5.7.32/sql/tc_log.cc:35
#15 0x00000000007d1937 in ha_commit_trans (thd=thd@entry=0x7fa67c018bc0, all=all@entry=false,
    ignore_global_read_lock=ignore_global_read_lock@entry=false)
    at /usr/src/mysql-5.7.32/sql/handler.cc:1807
#16 0x0000000000cf49e0 in trans_commit_stmt (thd=thd@entry=0x7fa67c018bc0)
    at /usr/src/mysql-5.7.32/sql/transaction.cc:465
#17 0x0000000000c4b22f in mysql_execute_command (thd=thd@entry=0x7fa67c018bc0,
    first_level=first_level@entry=true) at /usr/src/mysql-5.7.32/sql/sql_parse.cc:4995
#18 0x0000000000c4e2ad in mysql_parse (thd=thd@entry=0x7fa67c018bc0,
    parser_state=parser_state@entry=0x7fa6c1187740) at /usr/src/mysql-5.7.32/sql/sql_parse.cc:5584
#19 0x0000000000c4ed86 in dispatch_command (thd=thd@entry=0x7fa67c018bc0,
    com_data=com_data@entry=0x7fa6c1187da0, command=COM_QUERY)
    at /usr/src/mysql-5.7.32/sql/sql_parse.cc:1491
#20 0x0000000000c508c0 in do_command (thd=thd@entry=0x7fa67c018bc0)
    at /usr/src/mysql-5.7.32/sql/sql_parse.cc:1032
#21 0x0000000000d10ee0 in handle_connection (arg=arg@entry=0x6bc48e0)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:313
#22 0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b986f0)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#23 0x00007fa77baff6db in start_thread (arg=0x7fa6c1188700) at pthread_create.c:463
#24 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 35 (Thread 0x7fa6c11ca700 (LWP 9822)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>,
    mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>)
---Type <return> to continue, or q <return> to quit---
   c:655
#3  0x0000000000d10b1a in native_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection () at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6b971e0) at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b80450) at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6c11ca700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 34 (Thread 0x7fa6d8221700 (LWP 9811)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>, mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>) at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection () at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6b95830) at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6bc3da0) at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6d8221700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 33 (Thread 0x7fa6c124e700 (LWP 9729)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>, mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>) at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection () at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6bcaad0) at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6bc4290) at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6c124e700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 32 (Thread 0x7fa6c1356700 (LWP 9719)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>, mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>) at pthread_cond_wait.c:655
---Type <return> to continue, or q <return> to quit---
#3  0x0000000000d10b1a in native_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection () at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6b98100) at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6bc2d00) at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6c1356700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 31 (Thread 0x7fa6c15a8700 (LWP 9645)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>, mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>) at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection () at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6bc4590) at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b992d0) at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6c15a8700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 30 (Thread 0x7fa6d8095700 (LWP 9629)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>, mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>) at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection () at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6bc2b70) at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b95ea0) at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6d8095700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 29 (Thread 0x7fa770041700 (LWP 3972)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>, mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>) at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
---Type <return> to continue, or q <return> to quit---
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>, src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>) at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection () at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6b740b0) at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b74100) at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa770041700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 28 (Thread 0x7fa6c1ffb700 (LWP 3966)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x1d7df08 <COND_compress_gtid_table+40>) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x1d7df20 <LOCK_compress_gtid_table>, cond=0x1d7dee0 <COND_compress_gtid_table>) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x1d7dee0 <COND_compress_gtid_table>, mutex=mutex@entry=0x1d7df20 <LOCK_compress_gtid_table>) at pthread_cond_wait.c:655
#3  0x0000000000de8100 in native_cond_wait (mutex=<optimized out>, cond=<optimized out>) at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=<optimized out>, cond=<optimized out>) at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (that=<optimized out>, mutex=<optimized out>, src_file=0x154fcf8 "/usr/src/mysql-5.7.32/sql/rpl_gtid_persist.cc", src_line=885)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  compress_gtid_table (p_thd=p_thd@entry=0x6afb620) at /usr/src/mysql-5.7.32/sql/rpl_gtid_persist.cc:885
#7  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b56250) at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#8  0x00007fa77baff6db in start_thread (arg=0x7fa6c1ffb700) at pthread_create.c:463
#9  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 27 (Thread 0x7fa770083700 (LWP 3965)):
#0  0x00007fa779eed38c in __GI___sigtimedwait (set=<optimized out>, set@entry=0x7fa770082dd0, info=info@entry=0x7fa770082cb0, timeout=timeout@entry=0x0) at ../sysdeps/unix/sysv/linux/sigtimedwait.c:42
#1  0x00007fa77bb0a54c in __sigwait (set=set@entry=0x7fa770082dd0, sig=sig@entry=0x7fa770082d7c) at ../sysdeps/unix/sysv/linux/sigwait.c:28
#2  0x0000000000772f3b in signal_hand (arg=arg@entry=0x0) at /usr/src/mysql-5.7.32/sql/mysqld.cc:2131
#3  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b30a20) at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#4  0x00007fa77baff6db in start_thread (arg=0x7fa770083700) at pthread_create.c:463
#5  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 26 (Thread 0x7fa6c27fc700 (LWP 3964)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x30c75d8) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c7588, cond=0x30c75b0) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c75b0, mutex=mutex@entry=0x30c7588) at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c7578) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c7578, reset_sig_count=1, reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>, reset_sig_count=reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x00000000010fba8c in buf_resize_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0buf.cc:3010
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6c27fc700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 25 (Thread 0x7fa6c2ffd700 (LWP 3963)):
#0  0x00007fa77bb05fb9 in futex_reltimed_wait_cancelable (private=<optimized out>, reltime=0x7fa6c2ffc9c0, expected=0, futex_word=0x6a73158) at ../sysdeps/unix/sysv/linux/futex-internal.h:142
#1  __pthread_cond_wait_common (abstime=0x7fa6c2ffcbf0, mutex=0x6a73108, cond=0x6a73130) at pthread_cond_wait.c:533
#2  __pthread_cond_timedwait (cond=cond@entry=0x6a73130, mutex=mutex@entry=0x6a73108, abstime=abstime@entry=0x7fa6c2ffcbf0) at pthread_cond_wait.c:667
---Type <return> to continue, or q <return> to quit---
#3  0x0000000000fb02c3 in os_event::timed_wait (this=this@entry=0x6a730f8, abstime=abstime@entry=0x7fa6c2ffcbf0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:316
#4  0x0000000000fb0c31 in os_event::wait_time_low (this=0x6a730f8, time_in_usec=<optimized out>, reset_sig_count=1) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:488
#5  0x00000000010bbe85 in ib_wqueue_timedwait (wq=wq@entry=0x6a6a428, wait_in_usecs=wait_in_usecs@entry=5000000) at /usr/src/mysql-5.7.32/storage/innobase/ut/ut0wqueue.cc:166
#6  0x00000000011a96b3 in fts_optimize_thread (arg=0x6a6a428) at /usr/src/mysql-5.7.32/storage/innobase/fts/fts0opt.cc:2909
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6c2ffd700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 24 (Thread 0x7fa6c37fe700 (LWP 3962)):
#0  0x00007fa77bb05fb9 in futex_reltimed_wait_cancelable (private=<optimized out>, reltime=0x7fa6c37fdb00, expected=0, futex_word=0x6a0c728) at ../sysdeps/unix/sysv/linux/futex-internal.h:142
#1  __pthread_cond_wait_common (abstime=0x7fa6c37fdd30, mutex=0x6a0c6d8, cond=0x6a0c700) at pthread_cond_wait.c:533
#2  __pthread_cond_timedwait (cond=cond@entry=0x6a0c700, mutex=mutex@entry=0x6a0c6d8, abstime=abstime@entry=0x7fa6c37fdd30) at pthread_cond_wait.c:667
#3  0x0000000000fb02c3 in os_event::timed_wait (this=this@entry=0x6a0c6c8, abstime=abstime@entry=0x7fa6c37fdd30) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:316
#4  0x0000000000fb0c31 in os_event::wait_time_low (this=0x6a0c6c8, time_in_usec=<optimized out>, reset_sig_count=7) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:488
#5  0x000000000115d836 in dict_stats_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/dict/dict0stats_bg.cc:434
#6  0x00007fa77baff6db in start_thread (arg=0x7fa6c37fe700) at pthread_create.c:463
#7  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 23 (Thread 0x7fa6c3fff700 (LWP 3961)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x30c74b8) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c7468, cond=0x30c7490) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c7490, mutex=mutex@entry=0x30c7468) at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c7458) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c7458, reset_sig_count=1, reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>, reset_sig_count=reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x00000000011071ba in buf_dump_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0dump.cc:792
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6c3fff700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 22 (Thread 0x7fa6d8d25700 (LWP 3960)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x30c730c) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c72b8, cond=0x30c72e0) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c72e0, mutex=mutex@entry=0x30c72b8) at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c72a8) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c72a8, reset_sig_count=166, reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>, reset_sig_count=reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x0000000001057530 in srv_worker_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2535
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6d8d25700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 21 (Thread 0x7fa6d9526700 (LWP 3959)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x30c727c) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c7228, cond=0x30c7250) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c7250, mutex=mutex@entry=0x30c7228) at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c7218) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c7218, reset_sig_count=166, reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>, reset_sig_count=reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
---Type <return> to continue, or q <return> to quit---
#6  0x0000000001057530 in srv_worker_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2535
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6d9526700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 20 (Thread 0x7fa6d9d27700 (LWP 3958)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x30c71e8) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c7198, cond=0x30c71c0) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c71c0, mutex=mutex@entry=0x30c7198) at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c7188) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c7188, reset_sig_count=225, reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>, reset_sig_count=reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x0000000001057530 in srv_worker_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2535
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6d9d27700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 19 (Thread 0x7fa6da528700 (LWP 3957)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x30c715c) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c7108, cond=0x30c7130) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c7130, mutex=mutex@entry=0x30c7108) at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c70f8) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c70f8, reset_sig_count=reset_sig_count@entry=114) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>, reset_sig_count=reset_sig_count@entry=114) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x0000000001059780 in srv_purge_coordinator_suspend (rseg_history_len=<optimized out>, slot=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2691
#7  srv_purge_coordinator_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2807
#8  0x00007fa77baff6db in start_thread (arg=0x7fa6da528700) at pthread_create.c:463
#9  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 18 (Thread 0x7fa6e2cf6700 (LWP 3956)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x69eaafc) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x69eaaa8, cond=0x69eaad0) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x69eaad0, mutex=mutex@entry=0x69eaaa8) at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x69eaa98) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x69eaa98, reset_sig_count=26) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x0000000001068e1c in sync_array_wait_event (arr=arr@entry=0x303ca48, cell=@0x7fa6e2cf5c68: 0x7fa7702037e8) at /usr/src/mysql-5.7.32/storage/innobase/sync/sync0arr.cc:483
#7  0x000000000106b330 in rw_lock_s_lock_spin (lock=lock@entry=0x69ea5b0, pass=pass@entry=0, file_name=file_name@entry=0x1577690 "/usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc", line=line@entry=1746)
    at /usr/src/mysql-5.7.32/storage/innobase/sync/sync0rw.cc:433
#8  0x0000000000f834f6 in rw_lock_s_lock_func (pass=0, file_name=0x1577690 "/usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc", line=1746, lock=0x69ea5b0)
    at /usr/src/mysql-5.7.32/storage/innobase/include/sync0rw.ic:441
#9  pfs_rw_lock_s_lock_func (pass=0, file_name=0x1577690 "/usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc", line=1746, lock=0x69ea5b0) at /usr/src/mysql-5.7.32/storage/innobase/include/sync0rw.ic:804
#10 log_write_checkpoint_info (sync=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc:1746
#11 0x0000000000f83af0 in log_checkpoint (sync=sync@entry=true, write_always=write_always@entry=false) at /usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc:1892
#12 0x000000000105bf51 in srv_master_do_idle_tasks () at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2264
#13 srv_master_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2400
#14 0x00007fa77baff6db in start_thread (arg=0x7fa6e2cf6700) at pthread_create.c:463
#15 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
---Type <return> to continue, or q <return> to quit---

Thread 17 (Thread 0x7fa6e34f7700 (LWP 3955)):
#0  0x00007fa77bb05fb9 in futex_reltimed_wait_cancelable (private=<optimized out>, reltime=0x7fa6e34f6ba0, expected=0, futex_word=0x30c7428) at ../sysdeps/unix/sysv/linux/futex-internal.h:142
#1  __pthread_cond_wait_common (abstime=0x7fa6e34f6dd0, mutex=0x30c73d8, cond=0x30c7400) at pthread_cond_wait.c:533
#2  __pthread_cond_timedwait (cond=cond@entry=0x30c7400, mutex=mutex@entry=0x30c73d8, abstime=abstime@entry=0x7fa6e34f6dd0) at pthread_cond_wait.c:667
#3  0x0000000000fb02c3 in os_event::timed_wait (this=this@entry=0x30c73c8, abstime=abstime@entry=0x7fa6e34f6dd0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:316
#4  0x0000000000fb0c31 in os_event::wait_time_low (this=0x30c73c8, time_in_usec=<optimized out>, reset_sig_count=1) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:488
#5  0x0000000001056625 in srv_monitor_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:1592
#6  0x00007fa77baff6db in start_thread (arg=0x7fa6e34f7700) at pthread_create.c:463
#7  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 16 (Thread 0x7fa6e3cf8700 (LWP 3954)):
#0  0x00007fa77bb05fb9 in futex_reltimed_wait_cancelable (private=<optimized out>, reltime=0x7fa6e3cf7a30, expected=0, futex_word=0x30c7398) at ../sysdeps/unix/sysv/linux/futex-internal.h:142
#1  __pthread_cond_wait_common (abstime=0x7fa6e3cf7c60, mutex=0x30c7348, cond=0x30c7370) at pthread_cond_wait.c:533
#2  __pthread_cond_timedwait (cond=cond@entry=0x30c7370, mutex=mutex@entry=0x30c7348, abstime=abstime@entry=0x7fa6e3cf7c60) at pthread_cond_wait.c:667
#3  0x0000000000fb02c3 in os_event::timed_wait (this=this@entry=0x30c7338, abstime=abstime@entry=0x7fa6e3cf7c60) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:316
#4  0x0000000000fb0c31 in os_event::wait_time_low (this=0x30c7338, time_in_usec=<optimized out>, reset_sig_count=1) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:488
#5  0x000000000105a5f2 in srv_error_monitor_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:1758
#6  0x00007fa77baff6db in start_thread (arg=0x7fa6e3cf8700) at pthread_create.c:463
#7  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 15 (Thread 0x7fa6e44f9700 (LWP 3953)):
#0  0x00007fa77bb05fb9 in futex_reltimed_wait_cancelable (private=<optimized out>, reltime=0x7fa6e44f8af0, expected=0, futex_word=0x6a08d08) at ../sysdeps/unix/sysv/linux/futex-internal.h:142
#1  __pthread_cond_wait_common (abstime=0x7fa6e44f8d20, mutex=0x6a08cb8, cond=0x6a08ce0) at pthread_cond_wait.c:533
#2  __pthread_cond_timedwait (cond=cond@entry=0x6a08ce0, mutex=mutex@entry=0x6a08cb8, abstime=abstime@entry=0x7fa6e44f8d20) at pthread_cond_wait.c:667
#3  0x0000000000fb02c3 in os_event::timed_wait (this=this@entry=0x6a08ca8, abstime=abstime@entry=0x7fa6e44f8d20) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:316
#4  0x0000000000fb0c31 in os_event::wait_time_low (this=0x6a08ca8, time_in_usec=<optimized out>, reset_sig_count=1) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:488
#5  0x0000000000f7fcdb in lock_wait_timeout_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/lock/lock0wait.cc:497
#6  0x00007fa77baff6db in start_thread (arg=0x7fa6e44f9700) at pthread_create.c:463
#7  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 14 (Thread 0x7fa6db7a0700 (LWP 3951)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0, futex_word=0x6a09d3c) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x6a09ce8, cond=0x6a09d10) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x6a09d10, mutex=mutex@entry=0x6a09ce8) at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x6a09cd8) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x6a09cd8, reset_sig_count=64, reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>, reset_sig_count=reset_sig_count@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x000000000110ef97 in buf_flush_page_cleaner_worker (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:3507
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6db7a0700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 13 (Thread 0x7fa6dbfa1700 (LWP 3950)):
#0  0x00007fa77bb09bf7 in fsync (fd=fd@entry=9) at ../sysdeps/unix/sysv/linux/fsync.c:27
#1  0x0000000000fa5200 in os_file_fsync_posix (file=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:3087
#2  os_file_flush_func (file=file@entry=9) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:3191
---Type <return> to continue, or q <return> to quit---
#3  0x000000000116ca0d in pfs_os_file_flush_func (src_file=0x1598ed0 "/usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc", src_line=6043, file=...)
    at /usr/src/mysql-5.7.32/storage/innobase/include/os0file.ic:513
#4  fil_flush (space_id=space_id@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:6043
#5  0x00000000010ff0af in buf_dblwr_flush_buffered_writes () at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0dblwr.cc:1061
#6  0x000000000110d5c5 in buf_flush_end (flush_type=<optimized out>, buf_pool=0x33bc278) at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:1959
#7  buf_flush_do_batch (buf_pool=<optimized out>, type=<optimized out>, min_n=<optimized out>, lsn_limit=<optimized out>, n_processed=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:2028
#8  0x000000000110e3f2 in buf_flush_lists (min_n=100, lsn_limit=lsn_limit@entry=18446744073709551615, n_processed=0x7fa6dbfa0d08) at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:2132
#9  0x0000000001110c12 in buf_flush_page_cleaner_coordinator (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:3355
#10 0x00007fa77baff6db in start_thread (arg=0x7fa6dbfa1700) at pthread_create.c:463
#11 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 12 (Thread 0x7fa6dc7a2700 (LWP 3949)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6dc7a1d90) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6dc7a1d90, m1=m1@entry=0x7fa6dc7a1e30, m2=m2@entry=0x7fa6dc7a1e38, request=request@entry=0x7fa6dc7a1e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6dc7a1e40, m2=0x7fa6dc7a1e38, m1=0x7fa6dc7a1e30, global_segment=9) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=9, m1=m1@entry=0x7fa6dc7a1e30, m2=m2@entry=0x7fa6dc7a1e38, request=request@entry=0x7fa6dc7a1e40) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=9) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6dc7a2700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 11 (Thread 0x7fa6dcfa3700 (LWP 3948)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6dcfa2d90) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6dcfa2d90, m1=m1@entry=0x7fa6dcfa2e30, m2=m2@entry=0x7fa6dcfa2e38, request=request@entry=0x7fa6dcfa2e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6dcfa2e40, m2=0x7fa6dcfa2e38, m1=0x7fa6dcfa2e30, global_segment=8) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=8, m1=m1@entry=0x7fa6dcfa2e30, m2=m2@entry=0x7fa6dcfa2e38, request=request@entry=0x7fa6dcfa2e40) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=8) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6dcfa3700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 10 (Thread 0x7fa6dd7a4700 (LWP 3947)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6dd7a3d90) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6dd7a3d90, m1=m1@entry=0x7fa6dd7a3e30, m2=m2@entry=0x7fa6dd7a3e38, request=request@entry=0x7fa6dd7a3e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6dd7a3e40, m2=0x7fa6dd7a3e38, m1=0x7fa6dd7a3e30, global_segment=7) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=7, m1=m1@entry=0x7fa6dd7a3e30, m2=m2@entry=0x7fa6dd7a3e38, request=request@entry=0x7fa6dd7a3e40) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=7) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6dd7a4700) at pthread_create.c:463
---Type <return> to continue, or q <return> to quit---
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 9 (Thread 0x7fa6ddfa5700 (LWP 3946)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6ddfa4d90) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6ddfa4d90, m1=m1@entry=0x7fa6ddfa4e30, m2=m2@entry=0x7fa6ddfa4e38, request=request@entry=0x7fa6ddfa4e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6ddfa4e40, m2=0x7fa6ddfa4e38, m1=0x7fa6ddfa4e30, global_segment=6) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=6, m1=m1@entry=0x7fa6ddfa4e30, m2=m2@entry=0x7fa6ddfa4e38, request=request@entry=0x7fa6ddfa4e40) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=6) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6ddfa5700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 8 (Thread 0x7fa6de7a6700 (LWP 3945)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6de7a5d90) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6de7a5d90, m1=m1@entry=0x7fa6de7a5e30, m2=m2@entry=0x7fa6de7a5e38, request=request@entry=0x7fa6de7a5e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6de7a5e40, m2=0x7fa6de7a5e38, m1=0x7fa6de7a5e30, global_segment=5) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=5, m1=m1@entry=0x7fa6de7a5e30, m2=m2@entry=0x7fa6de7a5e38, request=request@entry=0x7fa6de7a5e40) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=5) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6de7a6700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 7 (Thread 0x7fa6defa7700 (LWP 3944)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6defa6d90) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6defa6d90, m1=m1@entry=0x7fa6defa6e30, m2=m2@entry=0x7fa6defa6e38, request=request@entry=0x7fa6defa6e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6defa6e40, m2=0x7fa6defa6e38, m1=0x7fa6defa6e30, global_segment=4) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=4, m1=m1@entry=0x7fa6defa6e30, m2=m2@entry=0x7fa6defa6e38, request=request@entry=0x7fa6defa6e40) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=4) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6defa7700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 6 (Thread 0x7fa6df7a8700 (LWP 3943)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6df7a7d90) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6df7a7d90, m1=m1@entry=0x7fa6df7a7e30, m2=m2@entry=0x7fa6df7a7e38, request=request@entry=0x7fa6df7a7e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6df7a7e40, m2=0x7fa6df7a7e38, m1=0x7fa6df7a7e30, global_segment=3) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=3, m1=m1@entry=0x7fa6df7a7e30, m2=m2@entry=0x7fa6df7a7e38, request=request@entry=0x7fa6df7a7e40) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=3) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
---Type <return> to continue, or q <return> to quit---
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6df7a8700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 5 (Thread 0x7fa6dffa9700 (LWP 3942)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6dffa8d90) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6dffa8d90, m1=m1@entry=0x7fa6dffa8e30, m2=m2@entry=0x7fa6dffa8e38, request=request@entry=0x7fa6dffa8e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6dffa8e40, m2=0x7fa6dffa8e38, m1=0x7fa6dffa8e30, global_segment=2) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=2, m1=m1@entry=0x7fa6dffa8e30, m2=m2@entry=0x7fa6dffa8e38, request=request@entry=0x7fa6dffa8e40) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=2) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6dffa9700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 4 (Thread 0x7fa6e07aa700 (LWP 3941)):
#0  0x00007fa77bb09bf7 in fsync (fd=fd@entry=3) at ../sysdeps/unix/sysv/linux/fsync.c:27
#1  0x0000000000fa5200 in os_file_fsync_posix (file=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:3087
#2  os_file_flush_func (file=file@entry=3) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:3191
#3  0x000000000116ca0d in pfs_os_file_flush_func (src_file=0x1598ed0 "/usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc", src_line=6043, file=...)
    at /usr/src/mysql-5.7.32/storage/innobase/include/os0file.ic:513
#4  fil_flush (space_id=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:6043
#5  0x0000000000f812d9 in log_io_complete (group=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc:943
#6  0x00000000011678f4 in fil_aio_wait (segment=segment@entry=1) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5901
#7  0x000000000105d428 in io_handler_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#8  0x00007fa77baff6db in start_thread (arg=0x7fa6e07aa700) at pthread_create.c:463
#9  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 3 (Thread 0x7fa6e0fab700 (LWP 3940)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6e0faad90) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6e0faad90, m1=m1@entry=0x7fa6e0faae30, m2=m2@entry=0x7fa6e0faae38, request=request@entry=0x7fa6e0faae40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6e0faae40, m2=0x7fa6e0faae38, m1=0x7fa6e0faae30, global_segment=0) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=0, m1=m1@entry=0x7fa6e0faae30, m2=m2@entry=0x7fa6e0faae38, request=request@entry=0x7fa6e0faae40) at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=0) at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6e0fab700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 2 (Thread 0x7fa771323700 (LWP 3939)):
#0  0x00007fa779eed38c in __GI___sigtimedwait (set=<optimized out>, set@entry=0x7fa771322d60, info=info@entry=0x7fa771322de0, timeout=timeout@entry=0x0) at ../sysdeps/unix/sysv/linux/sigtimedwait.c:42
#1  0x00007fa779eed407 in __GI___sigwaitinfo (set=set@entry=0x7fa771322d60, info=info@entry=0x7fa771322de0) at ../sysdeps/unix/sysv/linux/sigwaitinfo.c:25
#2  0x0000000000e8ea6b in timer_notify_thread_func (arg=arg@entry=0x7ffcfc88d750) at /usr/src/mysql-5.7.32/mysys/posix_timers.c:89
#3  0x0000000000eb8684 in pfs_spawn_thread (arg=0x305f730) at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
---Type <return> to continue, or q <return> to quit---
#4  0x00007fa77baff6db in start_thread (arg=0x7fa771323700) at pthread_create.c:463
#5  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 1 (Thread 0x7fa77bf29780 (LWP 3938)):
#0  0x00007fa779fc1cb9 in __GI___poll (fds=fds@entry=0x6ab1f30, nfds=2, timeout=timeout@entry=-1) at ../sysdeps/unix/sysv/linux/poll.c:29
#1  0x0000000000d12ab9 in Mysqld_socket_listener::listen_for_connection_event (this=0x6ab1ed0) at /usr/src/mysql-5.7.32/sql/conn_handler/socket_connection.cc:859
#2  0x000000000077a8e0 in Connection_acceptor<Mysqld_socket_listener>::connection_event_loop (this=0x6aae000) at /usr/src/mysql-5.7.32/sql/conn_handler/connection_acceptor.h:73
#3  mysqld_main (argc=<optimized out>, argv=<optimized out>) at /usr/src/mysql-5.7.32/sql/mysqld.cc:5132
#4  0x00007fa779ecebf7 in __libc_start_main (main=0x7580b0 <main(int, char**)>, argc=10, argv=0x7ffcfc88e258, init=<optimized out>, fini=<optimized out>, rtld_fini=<optimized out>, stack_end=0x7ffcfc88e248)
    at ../csu/libc-start.c:310
#5  0x00000000007700f4 in _start ()
``` 

结论: 发现线程卡在trx_flush_log_if_needed_low, 收到srv_flush_log_at_trx_commit参数的影响, 在需要刷redo log时发生排队

  - 只有同一文件的刷盘需要排队

需要观察: 

  1. log_write_flush_to_disk_low: redo log刷盘位置
  2. log_write_checkpoint_info: srv_master_do_idle_tasks: 写checkpoint信息
  3. buf_dblwr_flush_buffered_writes: buffer pool 刷盘

重新获取堆栈: 

```
(gdb) thread apply all bt

Thread 37 (Thread 0x7fa6e1faa700 (LWP 9840)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1db0f28 <Per_thread_connection_handler::COND_thread_cache+40>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (
    cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>,
    mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>)
    at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection ()
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6b993a0)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b80910)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6e1faa700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 36 (Thread 0x7fa6c1188700 (LWP 9833)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1db0f28 <Per_thread_connection_handler::COND_thread_cache+40>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at pthread_cond_wait.c:502
---Type <return> to continue, or q <return> to quit---
#2  __pthread_cond_wait (
    cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>,
    mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>)
    at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection ()
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6bc48e0)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b986f0)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6c1188700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 35 (Thread 0x7fa6c11ca700 (LWP 9822)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1db0f28 <Per_thread_connection_handler::COND_thread_cache+40>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (
    cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>,
    mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>)
    at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
---Type <return> to continue, or q <return> to quit---
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection ()
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6b971e0)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b80450)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6c11ca700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 34 (Thread 0x7fa6d8221700 (LWP 9811)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (
    cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>,
    mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>)
    at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection ()
---Type <return> to continue, or q <return> to quit---
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6b95830)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6bc3da0)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6d8221700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 33 (Thread 0x7fa6c124e700 (LWP 9729)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (
    cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>,
    mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>)
    at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection ()
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6bcaad0)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6bc4290)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6c124e700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 32 (Thread 0x7fa6c1356700 (LWP 9719)):
---Type <return> to continue, or q <return> to quit---
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (
    cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>,
    mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>)
    at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection ()
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6b98100)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6bc2d00)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6c1356700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 31 (Thread 0x7fa6c15a8700 (LWP 9645)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (
    cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>,
---Type <return> to continue, or q <return> to quit---
    mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>)
    at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection ()
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6bc4590)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b992d0)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6c15a8700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 30 (Thread 0x7fa6d8095700 (LWP 9629)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1db0f2c <Per_thread_connection_handler::COND_thread_cache+44>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (
    cond=cond@entry=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>,
    mutex=mutex@entry=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>)
    at pthread_cond_wait.c:655
#3  0x0000000000d10b1a in native_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    cond=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:202
---Type <return> to continue, or q <return> to quit---
#5  inline_mysql_cond_wait (
    mutex=0x1db0f40 <Per_thread_connection_handler::LOCK_thread_cache>,
    src_file=0x14e6390 "/usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc", src_line=152,
    that=0x1db0f00 <Per_thread_connection_handler::COND_thread_cache>)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  Per_thread_connection_handler::block_until_new_connection ()
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:152
#7  0x0000000000d10cf6 in handle_connection (arg=arg@entry=0x6bc2b70)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:344
#8  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b95ea0)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#9  0x00007fa77baff6db in start_thread (arg=0x7fa6d8095700) at pthread_create.c:463
#10 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 29 (Thread 0x7fa770041700 (LWP 3972)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x4edee78) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x4edee28, cond=0x4edee50)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x4edee50, mutex=mutex@entry=0x4edee28)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x4edee18)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x4edee18, reset_sig_count=1)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x0000000001068e1c in sync_array_wait_event (arr=arr@entry=0x303ca48,
    cell=@0x7fa77003d188: 0x7fa7702037e8)
    at /usr/src/mysql-5.7.32/storage/innobase/sync/sync0arr.cc:483
#7  0x00000000010ef12c in TTASEventMutex<BlockMutexPolicy>::wait (
    this=this@entry=0x7fa721e5fa18,
    filename=filename@entry=0x15911d8 "/usr/src/mysql-5.7.32/storage/innobase/buf/buf0buf.cc", line=line@entry=4751, spin=4)
    at /usr/src/mysql-5.7.32/storage/innobase/include/ut0mutex.ic:97
#8  0x00000000010ef41d in TTASEventMutex<BlockMutexPolicy>::spin_and_try_lock (
    line=<optimized out>,
    filename=0x15911d8 "/usr/src/mysql-5.7.32/storage/innobase/buf/buf0buf.cc",
    max_delay=6, max_spins=60, this=0x7fa721e5fa18)
    at /usr/src/mysql-5.7.32/storage/innobase/include/ib0mutex.h:858
#9  TTASEventMutex<BlockMutexPolicy>::enter (line=<optimized out>,
---Type <return> to continue, or q <return> to quit---
    filename=0x15911d8 "/usr/src/mysql-5.7.32/storage/innobase/buf/buf0buf.cc",
    max_delay=6, max_spins=<optimized out>, this=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/include/ib0mutex.h:715
#10 PolicyMutex<TTASEventMutex<BlockMutexPolicy> >::enter (
    this=this@entry=0x7fa721e5fa18, n_spins=n_spins@entry=30, n_delay=n_delay@entry=6,
    line=line@entry=4751,
    name=0x15911d8 "/usr/src/mysql-5.7.32/storage/innobase/buf/buf0buf.cc")
    at /usr/src/mysql-5.7.32/storage/innobase/include/ib0mutex.h:995
#11 0x00000000010f48ec in buf_page_get_known_nowait (rw_latch=rw_latch@entry=2,
    block=block@entry=0x7fa721e5f8c0, mode=mode@entry=51,
    file=file@entry=0x1590c38 "/usr/src/mysql-5.7.32/storage/innobase/btr/btr0sea.cc",
    line=line@entry=1037, mtr=mtr@entry=0x7fa77003ea50)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0buf.cc:4751
#12 0x00000000010e24dc in btr_search_guess_on_hash (index=index@entry=0x7fa6a093dbf8,
    info=info@entry=0x7fa6a093def8, tuple=tuple@entry=0x7fa62801ef88,
    mode=mode@entry=4, latch_mode=latch_mode@entry=2,
    cursor=cursor@entry=0x7fa77003e300, has_search_latch=<optimized out>,
    mtr=<optimized out>) at /usr/src/mysql-5.7.32/storage/innobase/btr/btr0sea.cc:1035
#13 0x00000000010d89b9 in btr_cur_search_to_nth_level (
    index=index@entry=0x7fa6a093dbf8, level=level@entry=0,
    tuple=tuple@entry=0x7fa62801ef88, mode=mode@entry=PAGE_CUR_LE,
    latch_mode=latch_mode@entry=2, cursor=cursor@entry=0x7fa77003e300,
    has_search_latch=0,
    file=0x157e730 "/usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc", line=2510,
    mtr=0x7fa77003ea50) at /usr/src/mysql-5.7.32/storage/innobase/btr/btr0cur.cc:931
#14 0x0000000000ff6bcb in btr_pcur_open_low (level=0,
    file=0x157e730 "/usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc",
    mtr=0x7fa77003ea50, line=2510, cursor=0x7fa77003e300, latch_mode=2,
    mode=PAGE_CUR_LE, tuple=0x7fa62801ef88, index=0x7fa6a093dbf8)
    at /usr/src/mysql-5.7.32/storage/innobase/include/btr0pcur.ic:472
#15 row_ins_clust_index_entry_low (flags=flags@entry=0, mode=mode@entry=2,
    index=index@entry=0x7fa6a093dbf8, n_uniq=n_uniq@entry=1,
    entry=entry@entry=0x7fa62801ef88, n_ext=n_ext@entry=0, thr=<optimized out>,
    dup_chk_only=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:2510
#16 0x0000000000ffa7a2 in row_ins_clust_index_entry (index=0x7fa6a093dbf8,
    entry=0x7fa62801ef88, thr=thr@entry=0x7fa6280051b0, n_ext=n_ext@entry=0,
    dup_chk_only=dup_chk_only@entry=false)
    at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:3299
#17 0x0000000000ffd10e in row_ins_index_entry (thr=0x7fa6280051b0,
    entry=<optimized out>, index=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:3437
#18 row_ins_index_entry_step (thr=0x7fa6280051b0, node=0x7fa628004f78)
---Type <return> to continue, or q <return> to quit---
    at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:3587
#19 row_ins (thr=0x7fa76bbff750, node=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:3725
#20 row_ins_step (thr=thr@entry=0x7fa6280051b0)
    at /usr/src/mysql-5.7.32/storage/innobase/row/row0ins.cc:3861
#21 0x000000000100d733 in row_insert_for_mysql_using_ins_graph (
    mysql_rec=mysql_rec@entry=0x7fa629a44c98 "\377%\r", prebuilt=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/row/row0mysql.cc:1746
#22 0x0000000001012194 in row_insert_for_mysql (
    mysql_rec=mysql_rec@entry=0x7fa629a44c98 "\377%\r", prebuilt=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/row/row0mysql.cc:1867
#23 0x0000000000f30db5 in ha_innobase::write_row (this=0x7fa629a449a0,
    record=0x7fa629a44c98 "\377%\r")
    at /usr/src/mysql-5.7.32/storage/innobase/handler/ha_innodb.cc:7627
#24 0x00000000007da018 in handler::ha_write_row (this=0x7fa629a449a0,
    buf=0x7fa629a44c98 "\377%\r") at /usr/src/mysql-5.7.32/sql/handler.cc:8107
#25 0x0000000000dc5a35 in write_record (thd=thd@entry=0x7fa6a0000d40,
    table=table@entry=0x7fa629a43ff0, info=info@entry=0x7fa77003f640,
    update=update@entry=0x7fa77003f6c0) at /usr/src/mysql-5.7.32/sql/sql_insert.cc:1895
#26 0x0000000000dc7401 in Sql_cmd_insert::mysql_insert (
    this=this@entry=0x7fa6a0005840, thd=thd@entry=0x7fa6a0000d40,
    table_list=table_list@entry=0x7fa6a00052b8)
    at /usr/src/mysql-5.7.32/sql/sql_insert.cc:776

#27 0x0000000000dc8172 in Sql_cmd_insert::execute (this=0x7fa6a0005840,
    thd=0x7fa6a0000d40) at /usr/src/mysql-5.7.32/sql/sql_insert.cc:3142
#28 0x0000000000c49531 in mysql_execute_command (thd=thd@entry=0x7fa6a0000d40,
    first_level=first_level@entry=true) at /usr/src/mysql-5.7.32/sql/sql_parse.cc:3606
#29 0x0000000000c4e2ad in mysql_parse (thd=thd@entry=0x7fa6a0000d40,
    parser_state=parser_state@entry=0x7fa770040740)
    at /usr/src/mysql-5.7.32/sql/sql_parse.cc:5584
#30 0x0000000000c4ed86 in dispatch_command (thd=thd@entry=0x7fa6a0000d40,
    com_data=com_data@entry=0x7fa770040da0, command=COM_QUERY)
    at /usr/src/mysql-5.7.32/sql/sql_parse.cc:1491
#31 0x0000000000c508c0 in do_command (thd=thd@entry=0x7fa6a0000d40)
    at /usr/src/mysql-5.7.32/sql/sql_parse.cc:1032
#32 0x0000000000d10ee0 in handle_connection (arg=arg@entry=0x6b740b0)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_handler_per_thread.cc:313
#33 0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b74100)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#34 0x00007fa77baff6db in start_thread (arg=0x7fa770041700) at pthread_create.c:463
#35 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 28 (Thread 0x7fa6c1ffb700 (LWP 3966)):
---Type <return> to continue, or q <return> to quit---
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x1d7df08 <COND_compress_gtid_table+40>)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0,
    mutex=0x1d7df20 <LOCK_compress_gtid_table>,
    cond=0x1d7dee0 <COND_compress_gtid_table>) at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x1d7dee0 <COND_compress_gtid_table>,
    mutex=mutex@entry=0x1d7df20 <LOCK_compress_gtid_table>) at pthread_cond_wait.c:655
#3  0x0000000000de8100 in native_cond_wait (mutex=<optimized out>,
    cond=<optimized out>) at /usr/src/mysql-5.7.32/include/thr_cond.h:147
#4  my_cond_wait (mp=<optimized out>, cond=<optimized out>)
    at /usr/src/mysql-5.7.32/include/thr_cond.h:202
#5  inline_mysql_cond_wait (that=<optimized out>, mutex=<optimized out>,
    src_file=0x154fcf8 "/usr/src/mysql-5.7.32/sql/rpl_gtid_persist.cc", src_line=885)
    at /usr/src/mysql-5.7.32/include/mysql/psi/mysql_thread.h:1187
#6  compress_gtid_table (p_thd=p_thd@entry=0x6afb620)
    at /usr/src/mysql-5.7.32/sql/rpl_gtid_persist.cc:885
#7  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b56250)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#8  0x00007fa77baff6db in start_thread (arg=0x7fa6c1ffb700) at pthread_create.c:463
#9  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 27 (Thread 0x7fa770083700 (LWP 3965)):
#0  0x00007fa779eed38c in __GI___sigtimedwait (set=<optimized out>,
    set@entry=0x7fa770082dd0, info=info@entry=0x7fa770082cb0,
    timeout=timeout@entry=0x0) at ../sysdeps/unix/sysv/linux/sigtimedwait.c:42
#1  0x00007fa77bb0a54c in __sigwait (set=set@entry=0x7fa770082dd0,
    sig=sig@entry=0x7fa770082d7c) at ../sysdeps/unix/sysv/linux/sigwait.c:28
#2  0x0000000000772f3b in signal_hand (arg=arg@entry=0x0)
    at /usr/src/mysql-5.7.32/sql/mysqld.cc:2131
#3  0x0000000000eb8684 in pfs_spawn_thread (arg=0x6b30a20)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#4  0x00007fa77baff6db in start_thread (arg=0x7fa770083700) at pthread_create.c:463
#5  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 26 (Thread 0x7fa6c27fc700 (LWP 3964)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x30c75d8) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c7588, cond=0x30c75b0)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c75b0, mutex=mutex@entry=0x30c7588)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c7578)
---Type <return> to continue, or q <return> to quit---
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c7578, reset_sig_count=1, reset_sig_count@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=reset_sig_count@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x00000000010fba8c in buf_resize_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0buf.cc:3010
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6c27fc700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 25 (Thread 0x7fa6c2ffd700 (LWP 3963)):
#0  0x00007fa77bb05fb9 in futex_reltimed_wait_cancelable (private=<optimized out>,
    reltime=0x7fa6c2ffc9c0, expected=0, futex_word=0x6a73158)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:142
#1  __pthread_cond_wait_common (abstime=0x7fa6c2ffcbf0, mutex=0x6a73108,
    cond=0x6a73130) at pthread_cond_wait.c:533
#2  __pthread_cond_timedwait (cond=cond@entry=0x6a73130, mutex=mutex@entry=0x6a73108,
    abstime=abstime@entry=0x7fa6c2ffcbf0) at pthread_cond_wait.c:667
#3  0x0000000000fb02c3 in os_event::timed_wait (this=this@entry=0x6a730f8,
    abstime=abstime@entry=0x7fa6c2ffcbf0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:316
#4  0x0000000000fb0c31 in os_event::wait_time_low (this=0x6a730f8,
    time_in_usec=<optimized out>, reset_sig_count=1)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:488
#5  0x00000000010bbe85 in ib_wqueue_timedwait (wq=wq@entry=0x6a6a428,
    wait_in_usecs=wait_in_usecs@entry=5000000)
    at /usr/src/mysql-5.7.32/storage/innobase/ut/ut0wqueue.cc:166
#6  0x00000000011a96b3 in fts_optimize_thread (arg=0x6a6a428)
    at /usr/src/mysql-5.7.32/storage/innobase/fts/fts0opt.cc:2909
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6c2ffd700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 24 (Thread 0x7fa6c37fe700 (LWP 3962)):
#0  0x00007fa77bb05fb9 in futex_reltimed_wait_cancelable (private=<optimized out>,
    reltime=0x7fa6c37fdb00, expected=0, futex_word=0x6a0c728)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:142
#1  __pthread_cond_wait_common (abstime=0x7fa6c37fdd30, mutex=0x6a0c6d8,
    cond=0x6a0c700) at pthread_cond_wait.c:533
#2  __pthread_cond_timedwait (cond=cond@entry=0x6a0c700, mutex=mutex@entry=0x6a0c6d8,
    abstime=abstime@entry=0x7fa6c37fdd30) at pthread_cond_wait.c:667
#3  0x0000000000fb02c3 in os_event::timed_wait (this=this@entry=0x6a0c6c8,
    abstime=abstime@entry=0x7fa6c37fdd30)
---Type <return> to continue, or q <return> to quit---
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:316
#4  0x0000000000fb0c31 in os_event::wait_time_low (this=0x6a0c6c8,
    time_in_usec=<optimized out>, reset_sig_count=7)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:488
#5  0x000000000115d836 in dict_stats_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/dict/dict0stats_bg.cc:434
#6  0x00007fa77baff6db in start_thread (arg=0x7fa6c37fe700) at pthread_create.c:463
#7  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 23 (Thread 0x7fa6c3fff700 (LWP 3961)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x30c74b8) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c7468, cond=0x30c7490)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c7490, mutex=mutex@entry=0x30c7468)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c7458)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c7458, reset_sig_count=1, reset_sig_count@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=reset_sig_count@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x00000000011071ba in buf_dump_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0dump.cc:792
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6c3fff700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 22 (Thread 0x7fa6d8d25700 (LWP 3960)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x30c730c) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c72b8, cond=0x30c72e0)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c72e0, mutex=mutex@entry=0x30c72b8)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c72a8)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c72a8, reset_sig_count=166, reset_sig_count@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=reset_sig_count@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x0000000001057530 in srv_worker_thread (arg=<optimized out>)
---Type <return> to continue, or q <return> to quit---
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2535
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6d8d25700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 21 (Thread 0x7fa6d9526700 (LWP 3959)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x30c727c) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c7228, cond=0x30c7250)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c7250, mutex=mutex@entry=0x30c7228)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c7218)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c7218, reset_sig_count=166, reset_sig_count@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=reset_sig_count@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x0000000001057530 in srv_worker_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2535
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6d9526700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 20 (Thread 0x7fa6d9d27700 (LWP 3958)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x30c71e8) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c7198, cond=0x30c71c0)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c71c0, mutex=mutex@entry=0x30c7198)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c7188)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c7188, reset_sig_count=225, reset_sig_count@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=reset_sig_count@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x0000000001057530 in srv_worker_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2535
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6d9d27700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 19 (Thread 0x7fa6da528700 (LWP 3957)):
---Type <return> to continue, or q <return> to quit---
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x30c715c) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x30c7108, cond=0x30c7130)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x30c7130, mutex=mutex@entry=0x30c7108)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x30c70f8)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x30c70f8, reset_sig_count=reset_sig_count@entry=114)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=reset_sig_count@entry=114)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x0000000001059780 in srv_purge_coordinator_suspend (
    rseg_history_len=<optimized out>, slot=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2691
#7  srv_purge_coordinator_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2807
#8  0x00007fa77baff6db in start_thread (arg=0x7fa6da528700) at pthread_create.c:463
#9  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 18 (Thread 0x7fa6e2cf6700 (LWP 3956)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x6a654fc) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x6a654a8, cond=0x6a654d0)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x6a654d0, mutex=mutex@entry=0x6a654a8)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x6a65498)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x6a65498, reset_sig_count=reset_sig_count@entry=20)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=reset_sig_count@entry=20)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x000000000116c954 in fil_flush (space_id=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:6025
#7  0x000000000116ccc4 in fil_flush_file_spaces (
    purpose=purpose@entry=FIL_TYPE_TABLESPACE)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:6136
#8  0x0000000000f83e52 in log_checkpoint (sync=sync@entry=true,
    write_always=write_always@entry=false)
    at /usr/src/mysql-5.7.32/storage/innobase/log/log0log.cc:1801
---Type <return> to continue, or q <return> to quit---
#9  0x000000000105bf51 in srv_master_do_idle_tasks ()
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2264
#10 srv_master_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:2400
#11 0x00007fa77baff6db in start_thread (arg=0x7fa6e2cf6700) at pthread_create.c:463
#12 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 17 (Thread 0x7fa6e34f7700 (LWP 3955)):
#0  0x00007fa77bb05fb9 in futex_reltimed_wait_cancelable (private=<optimized out>,
    reltime=0x7fa6e34f6ba0, expected=0, futex_word=0x30c7428)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:142
#1  __pthread_cond_wait_common (abstime=0x7fa6e34f6dd0, mutex=0x30c73d8,
    cond=0x30c7400) at pthread_cond_wait.c:533
#2  __pthread_cond_timedwait (cond=cond@entry=0x30c7400, mutex=mutex@entry=0x30c73d8,
    abstime=abstime@entry=0x7fa6e34f6dd0) at pthread_cond_wait.c:667
#3  0x0000000000fb02c3 in os_event::timed_wait (this=this@entry=0x30c73c8,
    abstime=abstime@entry=0x7fa6e34f6dd0)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:316
#4  0x0000000000fb0c31 in os_event::wait_time_low (this=0x30c73c8,
    time_in_usec=<optimized out>, reset_sig_count=1)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:488
#5  0x0000000001056625 in srv_monitor_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:1592
#6  0x00007fa77baff6db in start_thread (arg=0x7fa6e34f7700) at pthread_create.c:463
#7  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 16 (Thread 0x7fa6e3cf8700 (LWP 3954)):
#0  0x00007fa77bb05fb9 in futex_reltimed_wait_cancelable (private=<optimized out>,
    reltime=0x7fa6e3cf7a30, expected=0, futex_word=0x30c7398)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:142
#1  __pthread_cond_wait_common (abstime=0x7fa6e3cf7c60, mutex=0x30c7348,
    cond=0x30c7370) at pthread_cond_wait.c:533
#2  __pthread_cond_timedwait (cond=cond@entry=0x30c7370, mutex=mutex@entry=0x30c7348,
    abstime=abstime@entry=0x7fa6e3cf7c60) at pthread_cond_wait.c:667
#3  0x0000000000fb02c3 in os_event::timed_wait (this=this@entry=0x30c7338,
    abstime=abstime@entry=0x7fa6e3cf7c60)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:316
#4  0x0000000000fb0c31 in os_event::wait_time_low (this=0x30c7338,
    time_in_usec=<optimized out>, reset_sig_count=1)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:488
#5  0x000000000105a5f2 in srv_error_monitor_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0srv.cc:1758
#6  0x00007fa77baff6db in start_thread (arg=0x7fa6e3cf8700) at pthread_create.c:463
---Type <return> to continue, or q <return> to quit---
#7  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 15 (Thread 0x7fa6e44f9700 (LWP 3953)):
#0  0x00007fa77bb05fb9 in futex_reltimed_wait_cancelable (private=<optimized out>,
    reltime=0x7fa6e44f8af0, expected=0, futex_word=0x6a08d08)
    at ../sysdeps/unix/sysv/linux/futex-internal.h:142
#1  __pthread_cond_wait_common (abstime=0x7fa6e44f8d20, mutex=0x6a08cb8,
    cond=0x6a08ce0) at pthread_cond_wait.c:533
#2  __pthread_cond_timedwait (cond=cond@entry=0x6a08ce0, mutex=mutex@entry=0x6a08cb8,
    abstime=abstime@entry=0x7fa6e44f8d20) at pthread_cond_wait.c:667
#3  0x0000000000fb02c3 in os_event::timed_wait (this=this@entry=0x6a08ca8,
    abstime=abstime@entry=0x7fa6e44f8d20)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:316
#4  0x0000000000fb0c31 in os_event::wait_time_low (this=0x6a08ca8,
    time_in_usec=<optimized out>, reset_sig_count=1)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:488
#5  0x0000000000f7fcdb in lock_wait_timeout_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/lock/lock0wait.cc:497
#6  0x00007fa77baff6db in start_thread (arg=0x7fa6e44f9700) at pthread_create.c:463
#7  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 14 (Thread 0x7fa6db7a0700 (LWP 3951)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x6a123fc) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x6a123a8, cond=0x6a123d0)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x6a123d0, mutex=mutex@entry=0x6a123a8)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x6a12398)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x6a12398, reset_sig_count=reset_sig_count@entry=40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=reset_sig_count@entry=40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x00000000010feb5a in buf_dblwr_flush_buffered_writes ()
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0dblwr.cc:987
#7  0x000000000110d5c5 in buf_flush_end (flush_type=<optimized out>,
    buf_pool=0x33bbdb8) at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:1959
#8  buf_flush_do_batch (buf_pool=<optimized out>, type=<optimized out>,
    min_n=<optimized out>, lsn_limit=<optimized out>, n_processed=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:2028
#9  0x000000000110e997 in buf_flush_LRU_list (buf_pool=<optimized out>)
---Type <return> to continue, or q <return> to quit---
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:2300
#10 pc_flush_slot () at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:2865
#11 0x000000000110ef85 in buf_flush_page_cleaner_worker (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:3515
#12 0x00007fa77baff6db in start_thread (arg=0x7fa6db7a0700) at pthread_create.c:463
#13 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 13 (Thread 0x7fa6dbfa1700 (LWP 3950)):
#0  0x00007fa77bb05ad3 in futex_wait_cancelable (private=<optimized out>, expected=0,
    futex_word=0x4ecea88) at ../sysdeps/unix/sysv/linux/futex-internal.h:88
#1  __pthread_cond_wait_common (abstime=0x0, mutex=0x4ecea38, cond=0x4ecea60)
    at pthread_cond_wait.c:502
#2  __pthread_cond_wait (cond=cond@entry=0x4ecea60, mutex=mutex@entry=0x4ecea38)
    at pthread_cond_wait.c:655
#3  0x0000000000fb03e0 in os_event::wait (this=0x4ecea28)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:179
#4  os_event::wait_low (this=0x4ecea28, reset_sig_count=9)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:366
#5  0x0000000000fb064a in os_event_wait_low (event=<optimized out>,
    reset_sig_count=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0event.cc:611
#6  0x0000000001068e1c in sync_array_wait_event (arr=arr@entry=0x303ca48,
    cell=@0x7fa6dbfa0af8: 0x7fa770203168)
    at /usr/src/mysql-5.7.32/storage/innobase/sync/sync0arr.cc:483
#7  0x00000000011073bc in TTASEventMutex<GenericPolicy>::wait (
    this=this@entry=0x33bc278,
    filename=filename@entry=0x15932d8 "/usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc", line=line@entry=2278, spin=4)
    at /usr/src/mysql-5.7.32/storage/innobase/include/ut0mutex.ic:97
#8  0x000000000110ec4a in TTASEventMutex<GenericPolicy>::spin_and_try_lock (line=2278,
    filename=0x15932d8 "/usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc",
    max_delay=<optimized out>, max_spins=60, this=0x33bc278)
    at /usr/src/mysql-5.7.32/storage/innobase/include/ib0mutex.h:858
#9  TTASEventMutex<GenericPolicy>::enter (line=2278,
    filename=0x15932d8 "/usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc",
    max_delay=<optimized out>, max_spins=<optimized out>, this=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/include/ib0mutex.h:715
#10 PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (
    name=0x15932d8 "/usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc", line=2278,
    n_delay=<optimized out>, n_spins=<optimized out>, this=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/include/ib0mutex.h:995
#11 buf_flush_LRU_list (buf_pool=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:2278
---Type <return> to continue, or q <return> to quit---
#12 pc_flush_slot () at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:2865
#13 0x0000000001111005 in buf_flush_page_cleaner_coordinator (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:3309
#14 0x00007fa77baff6db in start_thread (arg=0x7fa6dbfa1700) at pthread_create.c:463
#15 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 12 (Thread 0x7fa6dc7a2700 (LWP 3949)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6dc7a1d90)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6dc7a1d90,
    m1=m1@entry=0x7fa6dc7a1e30, m2=m2@entry=0x7fa6dc7a1e38,
    request=request@entry=0x7fa6dc7a1e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6dc7a1e40,
    m2=0x7fa6dc7a1e38, m1=0x7fa6dc7a1e30, global_segment=9)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=9, m1=m1@entry=0x7fa6dc7a1e30,
    m2=m2@entry=0x7fa6dc7a1e38, request=request@entry=0x7fa6dc7a1e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=9)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6dc7a2700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 11 (Thread 0x7fa6dcfa3700 (LWP 3948)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6dcfa2d90)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6dcfa2d90,
    m1=m1@entry=0x7fa6dcfa2e30, m2=m2@entry=0x7fa6dcfa2e38,
    request=request@entry=0x7fa6dcfa2e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6dcfa2e40,
    m2=0x7fa6dcfa2e38, m1=0x7fa6dcfa2e30, global_segment=8)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=8, m1=m1@entry=0x7fa6dcfa2e30,
    m2=m2@entry=0x7fa6dcfa2e38, request=request@entry=0x7fa6dcfa2e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=8)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
---Type <return> to continue, or q <return> to quit---
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6dcfa3700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 10 (Thread 0x7fa6dd7a4700 (LWP 3947)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6dd7a3d90)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6dd7a3d90,
    m1=m1@entry=0x7fa6dd7a3e30, m2=m2@entry=0x7fa6dd7a3e38,
    request=request@entry=0x7fa6dd7a3e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6dd7a3e40,
    m2=0x7fa6dd7a3e38, m1=0x7fa6dd7a3e30, global_segment=7)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=7, m1=m1@entry=0x7fa6dd7a3e30,
    m2=m2@entry=0x7fa6dd7a3e38, request=request@entry=0x7fa6dd7a3e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=7)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6dd7a4700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 9 (Thread 0x7fa6ddfa5700 (LWP 3946)):
#0  0x00007fa77bb09bf7 in fsync (fd=fd@entry=26)
    at ../sysdeps/unix/sysv/linux/fsync.c:27
#1  0x0000000000fa5200 in os_file_fsync_posix (file=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:3087
#2  os_file_flush_func (file=file@entry=26)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:3191
#3  0x000000000116ca0d in pfs_os_file_flush_func (
    src_file=0x1598ed0 "/usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc",
    src_line=6043, file=...)
    at /usr/src/mysql-5.7.32/storage/innobase/include/os0file.ic:513
#4  fil_flush (space_id=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:6043
#5  0x000000000116ccc4 in fil_flush_file_spaces (
    purpose=purpose@entry=FIL_TYPE_TABLESPACE)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:6136
#6  0x00000000010fd3a1 in buf_dblwr_update (bpage=bpage@entry=0x7fa721e5f8c0,
---Type <return> to continue, or q <return> to quit---
    flush_type=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0dblwr.cc:758
#7  0x0000000001113ee2 in buf_flush_write_complete (bpage=bpage@entry=0x7fa721e5f8c0)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0flu.cc:812
#8  0x00000000010f511d in buf_page_io_complete (bpage=0x7fa721e5f8c0,
    evict=evict@entry=false)
    at /usr/src/mysql-5.7.32/storage/innobase/buf/buf0buf.cc:5820
#9  0x000000000116791f in fil_aio_wait (segment=segment@entry=6)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5896
#10 0x000000000105d428 in io_handler_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#11 0x00007fa77baff6db in start_thread (arg=0x7fa6ddfa5700) at pthread_create.c:463
#12 0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 8 (Thread 0x7fa6de7a6700 (LWP 3945)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6de7a5d90)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6de7a5d90,
    m1=m1@entry=0x7fa6de7a5e30, m2=m2@entry=0x7fa6de7a5e38,
    request=request@entry=0x7fa6de7a5e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6de7a5e40,
    m2=0x7fa6de7a5e38, m1=0x7fa6de7a5e30, global_segment=5)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=5, m1=m1@entry=0x7fa6de7a5e30,
    m2=m2@entry=0x7fa6de7a5e38, request=request@entry=0x7fa6de7a5e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=5)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6de7a6700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 7 (Thread 0x7fa6defa7700 (LWP 3944)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6defa6d90)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6defa6d90,
    m1=m1@entry=0x7fa6defa6e30, m2=m2@entry=0x7fa6defa6e38,
    request=request@entry=0x7fa6defa6e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
---Type <return> to continue, or q <return> to quit---
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6defa6e40,
    m2=0x7fa6defa6e38, m1=0x7fa6defa6e30, global_segment=4)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=4, m1=m1@entry=0x7fa6defa6e30,
    m2=m2@entry=0x7fa6defa6e38, request=request@entry=0x7fa6defa6e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=4)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6defa7700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 6 (Thread 0x7fa6df7a8700 (LWP 3943)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6df7a7d90)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6df7a7d90,
    m1=m1@entry=0x7fa6df7a7e30, m2=m2@entry=0x7fa6df7a7e38,
    request=request@entry=0x7fa6df7a7e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6df7a7e40,
    m2=0x7fa6df7a7e38, m1=0x7fa6df7a7e30, global_segment=3)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=3, m1=m1@entry=0x7fa6df7a7e30,
    m2=m2@entry=0x7fa6df7a7e38, request=request@entry=0x7fa6df7a7e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=3)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6df7a8700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 5 (Thread 0x7fa6dffa9700 (LWP 3942)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6dffa8d90)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6dffa8d90,
    m1=m1@entry=0x7fa6dffa8e30, m2=m2@entry=0x7fa6dffa8e38,
    request=request@entry=0x7fa6dffa8e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6dffa8e40,
---Type <return> to continue, or q <return> to quit---
    m2=0x7fa6dffa8e38, m1=0x7fa6dffa8e30, global_segment=2)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=2, m1=m1@entry=0x7fa6dffa8e30,
    m2=m2@entry=0x7fa6dffa8e38, request=request@entry=0x7fa6dffa8e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=2)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6dffa9700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 4 (Thread 0x7fa6e07aa700 (LWP 3941)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6e07a9d90)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6e07a9d90,
    m1=m1@entry=0x7fa6e07a9e30, m2=m2@entry=0x7fa6e07a9e38,
    request=request@entry=0x7fa6e07a9e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6e07a9e40,
    m2=0x7fa6e07a9e38, m1=0x7fa6e07a9e30, global_segment=1)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=1, m1=m1@entry=0x7fa6e07a9e30,
    m2=m2@entry=0x7fa6e07a9e38, request=request@entry=0x7fa6e07a9e40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=1)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6e07aa700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 3 (Thread 0x7fa6e0fab700 (LWP 3940)):
#0  0x00007fa77b8f663a in ?? () from /lib/x86_64-linux-gnu/libaio.so.1
#1  0x0000000000fa68d2 in LinuxAIOHandler::collect (this=this@entry=0x7fa6e0faad90)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2520
#2  0x0000000000fa72a2 in LinuxAIOHandler::poll (this=this@entry=0x7fa6e0faad90,
    m1=m1@entry=0x7fa6e0faae30, m2=m2@entry=0x7fa6e0faae38,
    request=request@entry=0x7fa6e0faae40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2680
#3  0x0000000000fae8dd in os_aio_linux_handler (request=0x7fa6e0faae40,
    m2=0x7fa6e0faae38, m1=0x7fa6e0faae30, global_segment=0)
---Type <return> to continue, or q <return> to quit---
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:2736
#4  os_aio_handler (segment=segment@entry=0, m1=m1@entry=0x7fa6e0faae30,
    m2=m2@entry=0x7fa6e0faae38, request=request@entry=0x7fa6e0faae40)
    at /usr/src/mysql-5.7.32/storage/innobase/os/os0file.cc:6282
#5  0x0000000001167860 in fil_aio_wait (segment=segment@entry=0)
    at /usr/src/mysql-5.7.32/storage/innobase/fil/fil0fil.cc:5862
#6  0x000000000105d428 in io_handler_thread (arg=<optimized out>)
    at /usr/src/mysql-5.7.32/storage/innobase/srv/srv0start.cc:319
#7  0x00007fa77baff6db in start_thread (arg=0x7fa6e0fab700) at pthread_create.c:463
#8  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 2 (Thread 0x7fa771323700 (LWP 3939)):
#0  0x00007fa779eed38c in __GI___sigtimedwait (set=<optimized out>,
    set@entry=0x7fa771322d60, info=info@entry=0x7fa771322de0,
    timeout=timeout@entry=0x0) at ../sysdeps/unix/sysv/linux/sigtimedwait.c:42
#1  0x00007fa779eed407 in __GI___sigwaitinfo (set=set@entry=0x7fa771322d60,
    info=info@entry=0x7fa771322de0) at ../sysdeps/unix/sysv/linux/sigwaitinfo.c:25
#2  0x0000000000e8ea6b in timer_notify_thread_func (arg=arg@entry=0x7ffcfc88d750)
    at /usr/src/mysql-5.7.32/mysys/posix_timers.c:89
#3  0x0000000000eb8684 in pfs_spawn_thread (arg=0x305f730)
    at /usr/src/mysql-5.7.32/storage/perfschema/pfs.cc:2197
#4  0x00007fa77baff6db in start_thread (arg=0x7fa771323700) at pthread_create.c:463
#5  0x00007fa779fce71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

Thread 1 (Thread 0x7fa77bf29780 (LWP 3938)):
#0  0x00007fa779fc1cb9 in __GI___poll (fds=fds@entry=0x6ab1f30, nfds=2,
    timeout=timeout@entry=-1) at ../sysdeps/unix/sysv/linux/poll.c:29
#1  0x0000000000d12ab9 in Mysqld_socket_listener::listen_for_connection_event (
    this=0x6ab1ed0) at /usr/src/mysql-5.7.32/sql/conn_handler/socket_connection.cc:859
#2  0x000000000077a8e0 in Connection_acceptor<Mysqld_socket_listener>::connection_event_loop (this=0x6aae000)
    at /usr/src/mysql-5.7.32/sql/conn_handler/connection_acceptor.h:73
#3  mysqld_main (argc=<optimized out>, argv=<optimized out>)
    at /usr/src/mysql-5.7.32/sql/mysqld.cc:5132
#4  0x00007fa779ecebf7 in __libc_start_main (main=0x7580b0 <main(int, char**)>,
    argc=10, argv=0x7ffcfc88e258, init=<optimized out>, fini=<optimized out>,
    rtld_fini=<optimized out>, stack_end=0x7ffcfc88e248) at ../csu/libc-start.c:310
#5  0x00000000007700f4 in _start ()
(gdb)
``` 

  - Thread 9: IO线程刷盘
    - 刷double write buffer: buf_dblwr_update
    - space_id = 37
  - Thread 13: buf_flush_page_cleaner_coordinator
    - 等待进入 buf_flush_LRU_list
  - Thread 14: buf_flush_page_cleaner_worker
    - 等待刷double write buffer
  - Thread 18: log_checkpoint
    - 等待space_id = 37刷盘
  - Thread 29: query
    - 等待buffer pool的相关page

初步结论: 

  - buffer pool 刷盘慢, 导致buffer pool中的block带锁, 以至于query 访问block 被卡住
