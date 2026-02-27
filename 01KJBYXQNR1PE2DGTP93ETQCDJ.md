---
title: 20230118 - 中移 crash
confluence_page_id: 2130452
created_at: 2023-01-18T16:09:40+00:00
updated_at: 2023-01-18T16:09:40+00:00
---

MySQL 8.0.30, 从 MySQL 8.0.27 升级上来, 运行了一段时间后崩溃, 重启仍然报错, 保留了数据现场

通过mysql-debug版本重启, 生成coredump, 对coredump进行分析

报错堆栈: 

```
#0  0x00007f80bc83aaa1 in pthread_kill () from /lib64/libpthread.so.0
#1  0x0000000003dfbb5a in my_write_core (sig=sig@entry=6) at ../../mysql-8.0.30/mysys/stacktrace.cc:295
#2  0x00000000030c7f96 in handle_fatal_signal (sig=6) at ../../mysql-8.0.30/sql/signal_handler.cc:202
#3  <signal handler called>
#4  0x00007f80baa76387 in raise () from /lib64/libc.so.6
#5  0x00007f80baa77a78 in abort () from /lib64/libc.so.6
#6  0x00000000030c806b in my_server_abort () at ../../mysql-8.0.30/sql/signal_handler.cc:258
#7  0x0000000003df5ff0 in my_abort () at ../../mysql-8.0.30/mysys/my_init.cc:258
#8  0x000000000411f0b0 in ut_dbg_assertion_failed (expr=expr@entry=0x475802f "!rec_new_is_versioned(rec)", file=file@entry=0x4f93b30 "../../../mysql-8.0.30/storage/innobase/rem/rem0rec.cc",
    line=<optimized out>, line@entry=1032) at ../../../mysql-8.0.30/storage/innobase/ut/ut0dbg.cc:99
#9  0x000000000402a4f3 in rec_convert_dtuple_to_rec_new (
    buf=buf@entry=0x7f75b000f6a0 "\016\016\t\t\017\t\006F\036\036e\023\016\025\t\017$?\377\377\377\377\377\255\067m\267\377\377\377\371\303$\202\200\300", index=index@entry=0x7f75ac5b4a78,
    dtuple=dtuple@entry=0x7f75b0010958) at ../../../mysql-8.0.30/storage/innobase/rem/rem0rec.cc:1032
#10 0x000000000402b9a5 in rec_convert_dtuple_to_rec (buf=0x7f75b000f6a0 "\016\016\t\t\017\t\006F\036\036e\023\016\025\t\017$?\377\377\377\377\377\255\067m\267\377\377\377\371\303$\202\200\300",
    index=index@entry=0x7f75ac5b4a78, dtuple=dtuple@entry=0x7f75b0010958) at ../../../mysql-8.0.30/storage/innobase/rem/rem0rec.cc:1064
#11 0x0000000004171112 in page_cur_tuple_insert (cursor=cursor@entry=0x7f75b0006e70, tuple=tuple@entry=0x7f75b0010958, index=0x7f75ac5b4a78, offsets=offsets@entry=0x7f7d86249d70,
    heap=heap@entry=0x7f7d86249d78, mtr=mtr@entry=0x7f7d86249d80) at ../../../mysql-8.0.30/storage/innobase/include/page0cur.ic:202
#12 0x000000000417147b in btr_cur_insert_if_possible (cursor=cursor@entry=0x7f75b0006e68, tuple=tuple@entry=0x7f75b0010958, offsets=offsets@entry=0x7f7d86249d70, heap=heap@entry=0x7f7d86249d78,
    mtr=mtr@entry=0x7f7d86249d80) at ../../../mysql-8.0.30/storage/innobase/btr/btr0cur.cc:2536
#13 0x0000000004175529 in btr_cur_pessimistic_update (flags=flags@entry=15, cursor=cursor@entry=0x7f75b0006e68, offsets=offsets@entry=0x7f7d86249d70, offsets_heap=offsets_heap@entry=0x7f7d86249d78,
    entry_heap=<optimized out>, big_rec=big_rec@entry=0x7f7d86249b40, update=0x7f75b0007c98, cmpl_info=0, thr=0x7f75b0006c18, trx_id=657378616, undo_no=135, mtr=0x7f7d86249d80, pcur=0x7f75b0006e68)
    at ../../../mysql-8.0.30/storage/innobase/btr/btr0cur.cc:3973
#14 0x000000000432168a in row_undo_mod_clust_low (node=node@entry=0x7f75b0006df8, offsets=offsets@entry=0x7f7d86249d70, offsets_heap=offsets_heap@entry=0x7f7d86249d78, heap=<optimized out>,
    heap@entry=0x7f75b00084c0, rebuilt_old_pk=rebuilt_old_pk@entry=0x7f7d86249d68, sys=sys@entry=0x7f7d86249d5b "", thr=0x7f75b0006c18, mtr=0x7f7d86249d80, mode=33)
    at ../../../mysql-8.0.30/storage/innobase/row/row0umod.cc:146
#15 0x000000000432284a in row_undo_mod_clust (node=node@entry=0x7f75b0006df8, thr=thr@entry=0x7f75b0006c18) at ../../../mysql-8.0.30/storage/innobase/row/row0umod.cc:308
#16 0x0000000004323b08 in row_undo_mod (node=node@entry=0x7f75b0006df8, thr=thr@entry=0x7f75b0006c18) at ../../../mysql-8.0.30/storage/innobase/row/row0umod.cc:1320
#17 0x0000000004098e32 in row_undo (node=node@entry=0x7f75b0006df8, thr=thr@entry=0x7f75b0006c18) at ../../../mysql-8.0.30/storage/innobase/row/row0undo.cc:301
#18 0x000000000409996a in row_undo_step (thr=thr@entry=0x7f75b0006c18) at ../../../mysql-8.0.30/storage/innobase/row/row0undo.cc:362
#19 0x000000000401f29d in que_thr_step (thr=thr@entry=0x7f75b0006c18) at ../../../mysql-8.0.30/storage/innobase/que/que0que.cc:913
#20 0x000000000401f490 in que_run_threads_low (thr=thr@entry=0x7f75b0006c18) at ../../../mysql-8.0.30/storage/innobase/que/que0que.cc:966
#21 0x000000000401f5f0 in que_run_threads (thr=thr@entry=0x7f75b0006c18) at ../../../mysql-8.0.30/storage/innobase/que/que0que.cc:1001
#22 0x00000000040feb9c in trx_rollback_active (trx=trx@entry=0x7f809d642818) at ../../../mysql-8.0.30/storage/innobase/trx/trx0roll.cc:624
#23 0x00000000040feef5 in trx_rollback_or_clean_resurrected (trx=trx@entry=0x7f809d642818, all=all@entry=true) at ../../../mysql-8.0.30/storage/innobase/trx/trx0roll.cc:691
#24 0x00000000040ff086 in trx_rollback_or_clean_recovered (all=all@entry=true) at ../../../mysql-8.0.30/storage/innobase/trx/trx0roll.cc:764
#25 0x00000000040ff231 in trx_recovery_rollback_thread () at ../../../mysql-8.0.30/storage/innobase/trx/trx0roll.cc:795
#26 0x0000000003fbf24d in std::__invoke_impl<void, void (*&)()> (__f=<optimized out>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:60
#27 0x0000000003fbf260 in std::__invoke<void (*&)()> (__fn=@0x7f7d8624ab80: 0x40ff199 <trx_recovery_rollback_thread()>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:95
#28 0x0000000003fbf26b in std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (this=this@entry=0x7f7d8624ab80, __args=empty std::tuple)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/functional:400
#29 0x0000000003fbf27f in std::_Bind<void (*())()>::operator()<, void>() (this=this@entry=0x7f7d8624ab80) at /opt/rh/devtoolset-8/root/usr/include/c++/8/functional:484
#30 0x0000000003fbf2fb in Detached_thread::operator()<void (*)()> (this=this@entry=0x7f75ac66c7f0, f=<optimized out>) at ../../../mysql-8.0.30/storage/innobase/include/os0thread-create.h:194
#31 0x0000000003fbf358 in std::__invoke_impl<void, Detached_thread, void (*)()> (__f=..., __args#0=@0x7f75ac66c7e8: 0x40ff199 <trx_recovery_rollback_thread()>)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:60
#32 0x0000000003fbf385 in std::__invoke<Detached_thread, void (*)()> (__fn=..., __args#0=@0x7f75ac66c7e8: 0x40ff199 <trx_recovery_rollback_thread()>)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/invoke.h:95
#33 0x0000000003fbfba5 in std::thread::_Invoker<std::tuple<Detached_thread, void (*)()> >::_M_invoke<0ul, 1ul> (this=<optimized out>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:244
#34 0x0000000003fbfbb3 in std::thread::_Invoker<std::tuple<Detached_thread, void (*)()> >::operator() (this=<optimized out>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/thread:253
``` 

断言错误: 

![image2023-1-18 23:38:51.png](/assets/01KJBYXQNR1PE2DGTP93ETQCDJ/image2023-1-18%2023%3A38%3A51.png)

操胜春 获得了稳定复现的场景. 此任务结束.
