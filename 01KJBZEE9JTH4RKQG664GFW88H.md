---
title: 20240816 - mysql crash 海南银行
confluence_page_id: 3146168
created_at: 2024-08-16T11:02:44+00:00
updated_at: 2024-08-20T05:58:08+00:00
---

# 背景

<https://support.actionsky.com/service_desk/browse/SHAI-12752>

堆栈: [zxhzldb_20240728.log.zip](/assets/01KJBZEE9JTH4RKQG664GFW88H/zxhzldb_20240728.log.zip)

用分离脚本分离每个线程的堆栈, 再进行排除:

[1.sh](/assets/01KJBZEE9JTH4RKQG664GFW88H/1.sh)

# 工单上的信息 排查

跟LOCK_status相关的线程: 多个THD::release_resources的线程, 都在等待 LOCK_status

(代码: [sql_class.cc](<http://sql_class.cc>):1111)

去除以上堆栈: 

```
grep -L 'sql_class.cc:1111'  * | xargs cat
``` 

跟status相关的线程, 只有一个, Thread 285: 

持有: LOCK_status (代码: PFS_status_variable_cache::do_materialize_global)

等待: srv_innodb_monitor_mutex (代码: srv_export_innodb_status)

```
Thread 285 (LWP 3295165):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc024050) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc024050, reset_sig_count=1) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdb2d030130: 0x7fde028497b0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x3ed4b00 <srv_innodb_monitor_mutex>, filename=filename@entry=0x30a1618 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc", line=line@entry=1529, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=this@entry=0x3ed4b00 <srv_innodb_monitor_mutex>, max_spins=60, max_spins@entry=30, max_delay=max_delay@entry=6, filename=filename@entry=0x30a1618 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc", line=line@entry=1529) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000021de567 in TTASEventMutex<GenericPolicy>::enter (line=1529, filename=0x30a1618 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc", max_delay=6, max_spins=30, this=0x3ed4b00 <srv_innodb_monitor_mutex>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (this=0x3ed4b00 <srv_innodb_monitor_mutex>, n_spins=30, n_delay=6, name=0x30a1618 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc", line=1529) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  0x000000000236b4b3 in mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=0x3ed4b00 <srv_innodb_monitor_mutex>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 srv_export_innodb_status () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:1529
#11 0x00000000021c436a in innodb_export_status () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/ha_innodb.cc:18441
#12 show_innodb_vars (thd=<optimized out>, var=0x7fdb2d031440, buff=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/ha_innodb.cc:21013
#13 0x000000000265560c in PFS_status_variable_cache::manifest (this=this@entry=0x7fda8445dc38, thd=0x7fdcbbca9a10, show_var_array=<optimized out>, status_vars=status_vars@entry=0x7fdb2d031d30, prefix=prefix@entry=0x28a91dc "", nested_array=nested_array@entry=false, strict=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs_variable.cc:1390
#14 0x00000000026564ea in PFS_status_variable_cache::do_materialize_global (this=0x7fda8445dc38) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs_variable.cc:1164
#15 0x000000000267dbbd in PFS_variable_cache<Status_variable>::materialize_global (this=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs_variable.h:521
#16 table_global_status::rnd_init (this=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/table_global_status.cc:122
#17 0x0000000002625089 in ha_perfschema::rnd_init (this=0x7fda844a0e08, scan=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/ha_perfschema.cc:1654
#18 0x00000000011304f9 in handler::ha_rnd_init (this=0x7fda844a0e08, scan=scan@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/handler.cc:2940
#19 0x0000000000e3022f in TableScanIterator::Init (this=0x7fda844a85e8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/records.cc:346
#20 0x000000000139a43b in MaterializeIterator::MaterializeQueryBlock (this=this@entry=0x7fda844a8638, query_block=..., stored_rows=stored_rows@entry=0x7fdb2d032558) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/composite_iterators.cc:817
#21 0x000000000139ae69 in MaterializeIterator::Init (this=0x7fda844a8638) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/composite_iterators.cc:632
#22 0x0000000000fb492d in Query_expression::ExecuteIteratorQuery (this=0x7fdcbbca3f68, thd=thd@entry=0x7fdcbbca9a10) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1224
#23 0x0000000000fb4bac in Query_expression::execute (this=<optimized out>, thd=0x7fdcbbca9a10) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1284
#24 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fdcbbca4a70, thd=thd@entry=0x7fdcbbca9a10) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#25 0x0000000000f565ff in Sql_cmd_show::execute (this=<optimized out>, thd=thd@entry=0x7fdcbbca9a10) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_show.cc:205
#26 0x0000000000f57edc in Sql_cmd_show_status::execute (this=<optimized out>, thd=0x7fdcbbca9a10) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_show.cc:748
#27 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fdcbbca9a10, first_level=first_level@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#28 0x0000000000ef6045 in dispatch_sql_command (thd=thd@entry=0x7fdcbbca9a10, parser_state=parser_state@entry=0x7fdb2d034640) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:5033
#29 0x0000000000ef79b5 in dispatch_command (thd=thd@entry=0x7fdcbbca9a10, com_data=com_data@entry=0x7fdb2d034d30, command=COM_QUERY) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1863
#30 0x0000000000ef921c in do_command (thd=thd@entry=0x7fdcbbca9a10) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1342
#31 0x000000000101d6f0 in handle_connection (arg=arg@entry=0x7309e60) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/conn_handler/connection_handler_per_thread.cc:301
#32 0x000000000262b551 in pfs_spawn_thread (arg=0x730b1e0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs.cc:2898
#33 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#34 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
``` 

去除以上堆栈: 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs cat
``` 

使用srv_innodb_monitor_mutex的函数较少, 找到持有方, Thread 21: 

持有srv_innodb_monitor_mutex, 等待trx_sys (代码: MVCC::size)

```
Thread 21 (LWP 3750803):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc329910) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc329910, reset_sig_count=2) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdb64716920: 0x7fde02846170) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fddfc32c958, filename=filename@entry=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=line@entry=697, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fddfc32c958, max_spins=60, max_delay=6, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=697) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000022df6be in TTASEventMutex<GenericPolicy>::enter (line=697, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", max_delay=6, max_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=697, name=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", n_delay=6, n_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 MVCC::size (this=0x7fddfc329b90) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc:697
#11 0x000000000236a7fc in srv_printf_innodb_monitor (file=<optimized out>, nowait=<optimized out>, trx_start_pos=trx_start_pos@entry=0x0, trx_end=trx_end@entry=0x0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:1440
#12 0x000000000236bed9 in srv_monitor_thread () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:1750
#13 0x000000000227c8d7 in std::__invoke_impl<void, void (*&)()> (__f=<optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:60
#14 std::__invoke<void (*&)()> (__fn=<synthetic pointer>: <optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:95
#15 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (__args=..., this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:467
#16 std::_Bind<void (*())()>::operator()<, void>() (this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:551
#17 Runnable::operator()<void (*)()> (f=@0x7fddfe6bdcc8: 0x236be20 <srv_monitor_thread()>, this=0x7fddfe6bdcd0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/os0thread-create.h:101
#18 std::__invoke_impl<void, Runnable, void (*)()> (__f=...) at /usr/include/c++/7.3.0/bits/invoke.h:60
#19 std::__invoke<Runnable, void (*)()> (__fn=...) at /usr/include/c++/7.3.0/bits/invoke.h:95
#20 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul> (this=0x7fddfe6bdcc8) at /usr/include/c++/7.3.0/thread:234
#21 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator() (this=0x7fddfe6bdcc8) at /usr/include/c++/7.3.0/thread:243
#22 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run (this=0x7fddfe6bdcc0) at /usr/include/c++/7.3.0/thread:186
#23 0x00007fde0ff4895a in ?? () from /usr/lib64/libstdc++.so.6
#24 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#25 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
``` 

去除以上堆栈: 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs grep -L 'read0read.cc:697' | xargs cat
``` 

去除AIO线程: 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs grep -L 'read0read.cc:697' | xargs grep -L 'fil_aio_wait'  | xargs cat
``` 

多个THD::release_resources的线程, 通过innobase_close_connection, 在等待trx_sys

```
Thread 73 (LWP 2882546):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc329910) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc329910, reset_sig_count=2) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdb60096a30: 0x7fde02848af0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fddfc32c958, filename=filename@entry=0x30a3d08 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0trx.cc", line=line@entry=621, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fddfc32c958, max_spins=60, max_delay=6, filename=0x30a3d08 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0trx.cc", line=621) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000023c641e in TTASEventMutex<GenericPolicy>::enter (line=621, filename=0x30a3d08 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0trx.cc", max_delay=6, max_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=621, name=0x30a3d08 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0trx.cc", n_delay=6, n_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 trx_disconnect_from_mysql (prepared=false, trx=0x7fde02446650) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0trx.cc:621
#11 trx_disconnect_plain (trx=0x7fde02446650) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0trx.cc:654
#12 trx_free_for_mysql (trx=trx@entry=0x7fde02446650) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0trx.cc:666
#13 0x00000000021e8e91 in innobase_close_connection (hton=<optimized out>, thd=0x7fdab8632c30) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/ha_innodb.cc:5930
#14 0x0000000001128c91 in closecon_handlerton (thd=0x7fdab8632c30, plugin=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/handler.cc:920
#15 0x0000000000f0fead in plugin_foreach_with_mask (thd=thd@entry=0x7fdab8632c30, funcs=funcs@entry=0x7fdb60096d20, type=type@entry=1, state_mask=4294967287, state_mask@entry=8, arg=arg@entry=0x0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_plugin.cc:2745
#16 0x0000000000f1012d in plugin_foreach_with_mask (thd=thd@entry=0x7fdab8632c30, func=func@entry=0x1128c50 <closecon_handlerton(THD*, plugin_ref, void*)>, type=type@entry=1, state_mask=state_mask@entry=8, arg=arg@entry=0x0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_plugin.cc:2758
#17 0x000000000112e16e in ha_close_connection (thd=thd@entry=0x7fdab8632c30) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/handler.cc:932
#18 0x0000000000e769b8 in THD::release_resources (this=this@entry=0x7fdab8632c30) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_class.cc:1085
#19 0x000000000101d675 in handle_connection (arg=arg@entry=0x6b29ad0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/conn_handler/connection_handler_per_thread.cc:308
#20 0x000000000262b551 in pfs_spawn_thread (arg=0x6d79690) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs.cc:2898
#21 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#22 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
``` 

去除相关线程 (根据锁上的文件位置): 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs grep -L 'read0read.cc:697' | xargs grep -L 'fil_aio_wait'  | xargs grep -L 'trx0trx.cc:621' | xargs cat
``` 

多个线程通过 trx_allocate_for_mysql在等待trx_sys, 去除相关线程: 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs grep -L 'read0read.cc:697' | xargs grep -L 'fil_aio_wait'  | xargs grep -L 'trx0trx.cc:621' | xargs grep -L 'trx0trx.cc:518' | xargs cat
``` 

去除等待命令的线程: 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs grep -L 'read0read.cc:697' | xargs grep -L 'fil_aio_wait'  | xargs grep -L 'trx0trx.cc:621' | xargs grep -L 'trx0trx.cc:518' | xargs grep -L 'Protocol_classic::get_command' | xargs cat
``` 

Thread 74: 遍历data_lock_waits表, 持有Global_exclusive_latch_guard, 等待trx_sys ([p_s.cc](<http://p_s.cc>):856)

```
Thread 74 (LWP 2894810):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc329910) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc329910, reset_sig_count=2) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdb4071da70: 0x7fde028460f0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fddfc32c958, filename=filename@entry=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", line=line@entry=856, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fddfc32c958, max_spins=60, max_delay=6, filename=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", line=856) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000022303d1 in TTASEventMutex<GenericPolicy>::enter (line=856, filename=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", max_delay=6, max_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=856, name=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", n_delay=6, n_spins=30, this=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 Innodb_data_lock_wait_iterator::scan (this=0x7fda60554750, container=0x7fdad5014fb0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc:856
#11 0x000000000265f72e in table_data_lock_waits::rnd_next (this=0x7fdad5014e60) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/table_data_lock_waits.cc:185
#12 0x0000000002624cae in ha_perfschema::rnd_next (this=0x7fdb5c0fe5a8, buf=0x7fdb5c0ffc38 "\377") at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/ha_perfschema.cc:1678
#13 0x00000000011306fc in handler::ha_rnd_next (this=0x7fdb5c0fe5a8, buf=0x7fdb5c0ffc38 "\377") at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/handler.cc:2985
#14 0x0000000000e305dd in TableScanIterator::Read (this=0x7fdad500fd70) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/records.cc:361
#15 0x0000000000fb4a6b in Query_expression::ExecuteIteratorQuery (this=0x7fdad500c208, thd=thd@entry=0x7fdad55ac090) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1231
#16 0x0000000000fb4bac in Query_expression::execute (this=<optimized out>, thd=0x7fdad55ac090) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1284
#17 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fdad500e248, thd=0x7fdad55ac090) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#18 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fdad55ac090, first_level=first_level@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#19 0x0000000000ef6045 in dispatch_sql_command (thd=thd@entry=0x7fdad55ac090, parser_state=parser_state@entry=0x7fdb4071f640) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:5033
#20 0x0000000000ef79b5 in dispatch_command (thd=thd@entry=0x7fdad55ac090, com_data=com_data@entry=0x7fdb4071fd30, command=COM_QUERY) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1863
#21 0x0000000000ef921c in do_command (thd=thd@entry=0x7fdad55ac090) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1342
#22 0x000000000101d6f0 in handle_connection (arg=arg@entry=0x6bdc4d0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/conn_handler/connection_handler_per_thread.cc:301
#23 0x000000000262b551 in pfs_spawn_thread (arg=0x6b48110) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs.cc:2898
#24 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#25 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
``` 

去除堆栈: 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs grep -L 'read0read.cc:697' | xargs grep -L 'fil_aio_wait'  | xargs grep -L 'trx0trx.cc:621' | xargs grep -L 'trx0trx.cc:518' | xargs grep -L 'Protocol_classic::get_command' | xargs grep -L 'data_lock_waits' | xargs cat
``` 

Thread 56: 执行CALL SQL, 等待trx_sys, 以获取水位线:

```
Thread 56 (LWP 2783865):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc329910) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc329910, reset_sig_count=2) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdbb009a220: 0x7fde02846070) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fddfc32c958, filename=filename@entry=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", line=line@entry=213, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=this@entry=0x7fddfc32c958, max_spins=60, max_spins@entry=30, max_delay=max_delay@entry=6, filename=filename@entry=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", line=line@entry=213) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000021de567 in TTASEventMutex<GenericPolicy>::enter (line=213, filename=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", max_delay=6, max_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (this=0x7fddfc32c958, n_spins=30, n_delay=6, name=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", line=213) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  0x0000000002256996 in mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 trx_rw_min_trx_id () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic:213
#11 lock_sec_rec_read_check_and_lock (duration=duration@entry=lock_duration_t::REGULAR, block=block@entry=0x7fddaa7ec730, rec=rec@entry=0x7fddab143cfb "0", index=index@entry=0x7fdb540455e8, offsets=0x7fdbb009ab10, sel_mode=sel_mode@entry=SELECT_ORDINARY, mode=LOCK_S, gap_mode=0, thr=0x7fdaf44f0420) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/lock/lock0lock.cc:5535
#12 0x00000000023447f5 in sel_set_rec_lock (pcur=pcur@entry=0x7fdaf44ef0e0, rec=rec@entry=0x7fddab143cfb "0", index=index@entry=0x7fdb540455e8, offsets=offsets@entry=0x7fdbb009ab10, sel_mode=SELECT_ORDINARY, mode=<optimized out>, type=0, thr=0x7fdaf44f0420, mtr=0x7fdbb009b150) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/row/row0sel.cc:1170
#13 0x0000000002349606 in row_search_mvcc (buf=buf@entry=0x7fdaf403aab8 "", mode=PAGE_CUR_GE, mode@entry=PAGE_CUR_UNSUPP, prebuilt=0x7fdaf44eeeb8, match_mode=match_mode@entry=1, direction=direction@entry=1) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/row/row0sel.cc:5225
#14 0x00000000021e9830 in ha_innobase::general_fetch (this=0x7fdaf40393f8, buf=0x7fdaf403aab8 "", direction=1, match_mode=1) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/ha_innodb.cc:10224
#15 0x0000000001131b5c in handler::ha_index_next_same (this=0x7fdaf40393f8, buf=0x7fdaf403aab8 "", key=0x7fdaf44f2870 "", keylen=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/handler.cc:3458
#16 0x0000000000eb6c0b in RefIterator<false>::Read (this=0x7fdaf44f2c00) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_executor.cc:3795
#17 0x0000000000fb4a6b in Query_expression::ExecuteIteratorQuery (this=0x7fdaf43e3280, thd=thd@entry=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1231
#18 0x0000000000fb4bac in Query_expression::execute (this=<optimized out>, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1284
#19 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fdaf43e6068, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#20 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fdaf405c460, first_level=first_level@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#21 0x0000000000e4b631 in sp_instr_stmt::exec_core (this=0x7fdaf43e61e8, thd=0x7fdaf405c460, nextp=0x7fdbb009cafc) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:966
#22 0x0000000000e4d699 in sp_lex_instr::reset_lex_and_exec_core (this=this@entry=0x7fdaf43e61e8, thd=thd@entry=0x7fdaf405c460, nextp=nextp@entry=0x7fdbb009cafc, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:441
#23 0x0000000000e4e6a4 in sp_lex_instr::validate_lex_and_execute_core (this=this@entry=0x7fdaf43e61e8, thd=thd@entry=0x7fdaf405c460, nextp=nextp@entry=0x7fdbb009cafc, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:727
#24 0x0000000000e4ff4c in sp_instr_stmt::execute (this=0x7fdaf43e61e8, thd=0x7fdaf405c460, nextp=0x7fdbb009cafc) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:892
#25 0x0000000000e44166 in sp_head::execute (this=this@entry=0x7fdaf40a0548, thd=thd@entry=0x7fdaf405c460, merge_da_on_success=merge_da_on_success@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2222
#26 0x0000000000e473f3 in sp_head::execute_procedure (this=this@entry=0x7fdaf40a0548, thd=thd@entry=0x7fdaf405c460, args=0x7fdaf4269da0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2871
#27 0x00000000013521e6 in Sql_cmd_call::execute_inner (this=0x7fdaf426a4e8, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_call.cc:235
#28 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fdaf426a4e8, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#29 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fdaf405c460, first_level=first_level@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#30 0x0000000000e4b631 in sp_instr_stmt::exec_core (this=0x7fdaf426a560, thd=0x7fdaf405c460, nextp=0x7fdbb009e59c) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:966
#31 0x0000000000e4d699 in sp_lex_instr::reset_lex_and_exec_core (this=this@entry=0x7fdaf426a560, thd=thd@entry=0x7fdaf405c460, nextp=nextp@entry=0x7fdbb009e59c, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:441
#32 0x0000000000e4e6a4 in sp_lex_instr::validate_lex_and_execute_core (this=this@entry=0x7fdaf426a560, thd=thd@entry=0x7fdaf405c460, nextp=nextp@entry=0x7fdbb009e59c, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:727
#33 0x0000000000e4ff4c in sp_instr_stmt::execute (this=0x7fdaf426a560, thd=0x7fdaf405c460, nextp=0x7fdbb009e59c) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:892
#34 0x0000000000e44166 in sp_head::execute (this=this@entry=0x7fdaf41592e8, thd=thd@entry=0x7fdaf405c460, merge_da_on_success=merge_da_on_success@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2222
#35 0x0000000000e473f3 in sp_head::execute_procedure (this=this@entry=0x7fdaf41592e8, thd=thd@entry=0x7fdaf405c460, args=0x7fdaf4469ca8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2871
#36 0x00000000013521e6 in Sql_cmd_call::execute_inner (this=0x7fdaf446a150, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_call.cc:235
#37 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fdaf446a150, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#38 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fdaf405c460, first_level=first_level@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#39 0x0000000000ef6045 in dispatch_sql_command (thd=thd@entry=0x7fdaf405c460, parser_state=parser_state@entry=0x7fdbb00a0640) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:5033
#40 0x0000000000ef79b5 in dispatch_command (thd=thd@entry=0x7fdaf405c460, com_data=com_data@entry=0x7fdbb00a0d30, command=COM_QUERY) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1863
#41 0x0000000000ef921c in do_command (thd=thd@entry=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1342
#42 0x000000000101d6f0 in handle_connection (arg=arg@entry=0x6df5380) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/conn_handler/connection_handler_per_thread.cc:301
#43 0x000000000262b551 in pfs_spawn_thread (arg=0x73076d0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs.cc:2898
#44 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#45 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
``` 

去除相关堆栈: 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs grep -L 'read0read.cc:697' | xargs grep -L 'fil_aio_wait'  | xargs grep -L 'trx0trx.cc:621' | xargs grep -L 'trx0trx.cc:518' | xargs grep -L 'Protocol_classic::get_command' | xargs grep -L 'data_lock_waits' | xargs grep -L 'Sql_cmd_call' | xargs cat
``` 

purge_coordinator_thread, 持有了purge_sys->latch, 等待trx_sys:

```
Thread 32 (LWP 3750817):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc329910) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc329910, reset_sig_count=2) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdb43ffea60: 0x7fde02846130) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fddfc32c958, filename=filename@entry=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=line@entry=671, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fddfc32c958, max_spins=60, max_delay=6, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=671) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000022dfa9e in TTASEventMutex<GenericPolicy>::enter (line=671, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", max_delay=<optimized out>, max_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=671, name=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", n_delay=<optimized out>, n_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 MVCC::clone_oldest_view (this=0x7fddfc329b90, view=0x7fddfe6780a0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc:671
#11 0x0000000002399314 in trx_purge (n_purge_threads=4, batch_size=300, truncate=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0purge.cc:2409
#12 0x000000000236f196 in srv_do_purge (n_total_purged=<synthetic pointer>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:2923
#13 srv_purge_coordinator_thread () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:3101
#14 0x000000000227c8d7 in std::__invoke_impl<void, void (*&)()> (__f=<optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:60
#15 std::__invoke<void (*&)()> (__fn=<synthetic pointer>: <optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:95
#16 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (__args=..., this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:467
#17 std::_Bind<void (*())()>::operator()<, void>() (this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:551
#18 Runnable::operator()<void (*)()> (f=@0x6b6df78: 0x236ece0 <srv_purge_coordinator_thread()>, this=0x6b6df80) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/os0thread-create.h:101
#19 std::__invoke_impl<void, Runnable, void (*)()> (__f=...) at /usr/include/c++/7.3.0/bits/invoke.h:60
#20 std::__invoke<Runnable, void (*)()> (__fn=...) at /usr/include/c++/7.3.0/bits/invoke.h:95
#21 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul> (this=0x6b6df78) at /usr/include/c++/7.3.0/thread:234
#22 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator() (this=0x6b6df78) at /usr/include/c++/7.3.0/thread:243
#23 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run (this=0x6b6df70) at /usr/include/c++/7.3.0/thread:186
#24 0x00007fde0ff4895a in ?? () from /usr/lib64/libstdc++.so.6
#25 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#26 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
``` 

去除相关堆栈: 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs grep -L 'read0read.cc:697' | xargs grep -L 'fil_aio_wait'  | xargs grep -L 'trx0trx.cc:621' | xargs grep -L 'trx0trx.cc:518' | xargs grep -L 'Protocol_classic::get_command' | xargs grep -L 'data_lock_waits' | xargs grep -L 'Sql_cmd_call' | xargs grep -L 'trx_purge' | xargs cat
``` 

error_monitor_thread, 需要refresh_innodb_monitor_stats, 等待srv_innodb_monitor_mutex

```
Thread 20 (LWP 3750802):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc024050) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc024050, reset_sig_count=1) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdb63714a60: 0x7fde02846030) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x3ed4b00 <srv_innodb_monitor_mutex>, filename=filename@entry=0x30a1618 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc", line=line@entry=1250, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x3ed4b00 <srv_innodb_monitor_mutex>, max_spins=60, max_delay=6, filename=0x30a1618 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc", line=1250) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x000000000236c72c in TTASEventMutex<GenericPolicy>::enter (line=1250, filename=0x30a1618 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc", max_delay=6, max_spins=30, this=0x3ed4b00 <srv_innodb_monitor_mutex>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=1250, name=0x30a1618 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc", n_delay=6, n_spins=30, this=0x3ed4b00 <srv_innodb_monitor_mutex>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=0x3ed4b00 <srv_innodb_monitor_mutex>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 srv_refresh_innodb_monitor_stats () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:1250
#11 srv_error_monitor_thread () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:1821
#12 0x000000000227c8d7 in std::__invoke_impl<void, void (*&)()> (__f=<optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:60
#13 std::__invoke<void (*&)()> (__fn=<synthetic pointer>: <optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:95
#14 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (__args=..., this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:467
#15 std::_Bind<void (*())()>::operator()<, void>() (this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:551
#16 Runnable::operator()<void (*)()> (f=@0x7fddfe6bdc08: 0x236c4c0 <srv_error_monitor_thread()>, this=0x7fddfe6bdc10) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/os0thread-create.h:101
#17 std::__invoke_impl<void, Runnable, void (*)()> (__f=...) at /usr/include/c++/7.3.0/bits/invoke.h:60
#18 std::__invoke<Runnable, void (*)()> (__fn=...) at /usr/include/c++/7.3.0/bits/invoke.h:95
#19 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul> (this=0x7fddfe6bdc08) at /usr/include/c++/7.3.0/thread:234
#20 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator() (this=0x7fddfe6bdc08) at /usr/include/c++/7.3.0/thread:243
#21 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run (this=0x7fddfe6bdc00) at /usr/include/c++/7.3.0/thread:186
#22 0x00007fde0ff4895a in ?? () from /usr/lib64/libstdc++.so.6
#23 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#24 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
``` 

去除相关堆栈: 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs grep -L 'read0read.cc:697' | xargs grep -L 'fil_aio_wait'  | xargs grep -L 'trx0trx.cc:621' | xargs grep -L 'trx0trx.cc:518' | xargs grep -L 'Protocol_classic::get_command' | xargs grep -L 'data_lock_waits' | xargs grep -L 'Sql_cmd_call' | xargs grep -L 'trx_purge' | xargs grep -L 'srv_refresh_innodb_monitor_stats' | xargs cat
``` 

剩余以下线程: 

```
grep -L 'sql_class.cc:1111'  * | xargs grep -L 'do_materialize_global' | xargs grep -L 'read0read.cc:697' | xargs grep -L 'fil_aio_wait'  | xargs grep -L 'trx0trx.cc:621' | xargs grep -L 'trx0trx.cc:518' | xargs grep -L 'Protocol_classic::get_command' | xargs grep -L 'data_lock_waits' | xargs grep -L 'Sql_cmd_call' | xargs grep -L 'trx_purge' | xargs grep -L 'srv_refresh_innodb_monitor_stats'
thread_1.txt: Connection_acceptor线程, 等待新连接
thread_12.txt: buf_flush_page_coordinator_thread线程, 睡眠
thread_13.txt: buf_flush_page_cleaner_thread线程, 睡眠
thread_14.txt: log_checkpointer线程, 睡眠
thread_15.txt: log_flush_notifier线程, 睡眠
thread_16.txt: log_flusher线程, 睡眠
thread_17.txt: log_write_notifier线程, 睡眠
thread_18.txt: log_writer线程, 睡眠
thread_19.txt: lock_wait_timeout_thread线程, 睡眠
thread_22.txt: buf_resize_thread线程, 睡眠
thread_23.txt: master_thread线程, 睡眠
thread_24.txt: dict_stats_thread线程, 睡眠
thread_25.txt: fts_optimize_thread线程, 睡眠
thread_26.txt: Scheduler_dynamic::worker线程, 睡眠
thread_27.txt: Scheduler_dynamic::worker线程, 睡眠
thread_28.txt: ngs Server, 睡眠
thread_29.txt: 半同步 Ack_receiver, 睡眠
thread_30.txt: buf_dump_thread, 睡眠
thread_31.txt: Clone_persist_gtid::periodic_write, 睡眠
thread_33.txt: innodb srv_worker_thread, 睡眠
thread_34.txt: innodb srv_worker_thread, 睡眠: 
thread_35.txt: innodb srv_worker_thread, 睡眠: 
thread_36.txt: signal handler, 睡眠
thread_37.txt: ngs::Server, 睡眠
thread_38.txt: compress_gtid_table, 等待COND_compress_gtid_table
thread_39.txt: com_binlog_dump, 等待binlog 的 update_cond (LOCK_binlog_end_pos)
``` 

线程关系的简述: 

```
跟LOCK_status相关的线程: 多个THD::release_resources的线程, 都在等待 LOCK_status

---

error_monitor_thread, 需要refresh_innodb_monitor_stats, 等待srv_innodb_monitor_mutex

Thread 285: 持有: LOCK_status, 等待: srv_innodb_monitor_mutex (代码: srv_export_innodb_status)

---

Thread 21: printf_innodb_monitor, 持有srv_innodb_monitor_mutex, 等待trx_sys (代码: MVCC::size)

purge_coordinator_thread, 持有了purge_sys->latch, 等待trx_sys:

Thread 56: 执行CALL SQL, 等待trx_sys, 以获取水位线. processlist ID=553750

Thread 74: 遍历data_lock_waits表, 持有Global_exclusive_latch_guard, 等待trx_sys (p_s.cc:856); processlist ID=553852

多个线程通过 trx_allocate_for_mysql在等待trx_sys

多个THD::release_resources的线程, 通过innobase_close_connection, 在等待trx_sys

``` 

一次完整的InnoDB Monitor: 

```
InnoDB: ###### Starts InnoDB Monitor for 30 secs to print diagnostic info:
InnoDB: Pending preads 0, pwrites 0

=====================================
2024-07-28 07:09:24 140580259723008 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 66 seconds
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 226758 srv_active, 0 srv_shutdown, 4919680 srv_idle
srv_master_thread log flush and writes: 0
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 8163271
--Thread 140581528016640 has waited at trx0sys.ic line 213 for 227 seconds the semaphore:
Mutex at 0x7fddfc32c958, Mutex TRX_SYS created trx0sys.cc:565, lock var 1

--Thread 140581530375936 has waited at trx0trx.cc line 621 for 133 seconds the semaphore:
Mutex at 0x7fddfc32c958, Mutex TRX_SYS created trx0sys.cc:565, lock var 1

--Thread 140579655780096 has waited at p_s.cc line 856 for 226 seconds the semaphore:
Mutex at 0x7fddfc32c958, Mutex TRX_SYS created trx0sys.cc:565, lock var 1

--Thread 140579715413760 has waited at read0read.cc line 671 for 249 seconds the semaphore:
Mutex at 0x7fddfc32c958, Mutex TRX_SYS created trx0sys.cc:565, lock var 1

--Thread 140579655190272 has waited at trx0trx.cc line 518 for 98 seconds the semaphore:
Mutex at 0x7fddfc32c958, Mutex TRX_SYS created trx0sys.cc:565, lock var 1

OS WAIT ARRAY INFO: signal count 8407983
RW-shared spins 0, rounds 0, OS waits 0
RW-excl spins 0, rounds 0, OS waits 0
RW-sx spins 0, rounds 0, OS waits 0
Spin rounds per wait: 0.00 RW-shared, 0.00 RW-excl, 0.00 RW-sx
FAIL TO OBTAIN LOCK MUTEX, SKIP LOCK INFO PRINTING
--------
FILE I/O
--------
I/O thread 0 state: waiting for i/o request (insert buffer thread)
I/O thread 1 state: waiting for i/o request (log thread)
I/O thread 2 state: waiting for i/o request (read thread)
I/O thread 3 state: waiting for i/o request (read thread)
I/O thread 4 state: waiting for i/o request (read thread)
I/O thread 5 state: waiting for i/o request (read thread)
I/O thread 6 state: waiting for i/o request (write thread)
I/O thread 7 state: waiting for i/o request (write thread)
I/O thread 8 state: waiting for i/o request (write thread)
I/O thread 9 state: waiting for i/o request (write thread)
Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
 ibuf aio reads:, log i/o's:, sync i/o's:
Pending flushes (fsync) log: 0; buffer pool: 854
50973788 OS file reads, 799237878 OS file writes, 57327292 OS fsyncs
0.00 reads/s, 0 avg bytes/read, 0.00 writes/s, 0.00 fsyncs/s
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 57485, seg size 57487, 88554 merges
merged operations:
 insert 1189319, delete mark 6154138, delete 3
discarded operations:
 insert 0, delete mark 0, delete 0
Hash table size 2212699, node heap has 28 buffer(s)
Hash table size 2212699, node heap has 92 buffer(s)
Hash table size 2212699, node heap has 14723 buffer(s)
Hash table size 2212699, node heap has 1 buffer(s)
Hash table size 2212699, node heap has 1 buffer(s)
Hash table size 2212699, node heap has 2 buffer(s)
Hash table size 2212699, node heap has 5 buffer(s)
Hash table size 2212699, node heap has 3 buffer(s)
0.00 hash searches/s, 0.00 non-hash searches/s
---
LOG
---
Log sequence number          1899955116118
Log buffer assigned up to    1899955116118
Log buffer completed up to   1899955116118
Log written up to            1899955116118
Log flushed up to            1899955116118
Added dirty pages up to      1899955116118
Pages flushed up to          1899955116118
Last checkpoint at           1899955116118
616632341 log i/o's done, 0.00 log i/o's/second
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 8770551808
Dictionary memory allocated 1623048
Buffer pool size   524241
Free buffers       2048
Database pages     505922
Old database pages 186715
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 29807857, not young 2348664861
0.00 youngs/s, 0.00 non-youngs/s
Pages read 50946715, created 103268285, written 146779585
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 505922, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
----------------------
INDIVIDUAL BUFFER POOL INFO
----------------------
---BUFFER POOL 0
Buffer pool size   262129
Free buffers       1024
Database pages     252970
Old database pages 93361
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 14936244, not young 1163296589
0.00 youngs/s, 0.00 non-youngs/s
Pages read 25447186, created 51632387, written 73383618
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 252970, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 1
Buffer pool size   262112
Free buffers       1024
Database pages     252952
Old database pages 93354
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 14871613, not young 1185368272
0.00 youngs/s, 0.00 non-youngs/s
Pages read 25499529, created 51635898, written 73395967
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 252952, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue

 
(此处断开, 也就是Thread 21, 等待在 srv0srv.cc:1440)
``` 
    
    
    日志最后断开, 也就是Thread 21, 等待在 srv0srv.cc:1440

其中"FAIL TO OBTAIN LOCK MUTEX, SKIP LOCK INFO PRINTING", 是因为Global_exclusive_try_latch获取失败

完整的processlist: 

```
BHNDB> show processlist;
+--------+----------+--------------------+------+-------------+---------+----------------------------------------------------------------------------------------------------------------------------------------+
| Id     | User     | Host               | db   | Command     | Time    | State                                                                                                                                  |
+--------+----------+--------------------+------+-------------+---------+----------------------------------------------------------------------------------------------------------------------------------------+
|  73826 | repl     | 10.21.18.163:58992 | NULL | Binlog Dump | 4419396 | Source has sent all binlog to replica; waiting for more                                                                                |
| 553750 | hnyhfxjd | 10.21.18.162:34832 | fxjd | Query       |   45112 | executing                                              L_TRUE
        SELECT * FROM CBS_MDM_AC_REL L WHERE L.NOTE_STS = '0'                  |
| 553852 | igcam    | localhost          | NULL | Query       |   44546 | executing                                              ance_schema.data_lock_waits                                                     |
| 553865 | hnyhfxjd | 10.21.18.161:51358 | fxjd | Query       |   44417 | Opening tables                                         5_0_, this_.CODE as CODE5_0_, this_.NAME as NAME5_0_, this_.FILE_HTTP_IP as FIL |
| 554170 | hnyhfxjd | 10.21.18.160:58560 | fxjd | Prepare     |   42627 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 554211 | hnyhfxjd | 10.21.18.160:58764 | fxjd | Prepare     |   42405 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 554264 | hnyhfxjd | 10.21.18.160:59032 | fxjd | Query       |   42107 | Opening tables                                         5_0_, this_.CODE as CODE5_0_, this_.NAME as NAME5_0_, this_.FILE_HTTP_IP as FIL |
| 554315 | hnyhfxjd | 10.21.18.160:59290 | fxjd | Prepare     |   41812 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 554436 | hnyhfxjd | 10.21.18.160:59910 | fxjd | Prepare     |   41074 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 554497 | hnyhfxjd | 10.21.18.160:60226 | fxjd | Prepare     |   40701 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 554827 | hnyhfxjd | 10.21.18.160:33688 | fxjd | Prepare     |   38745 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 554838 | hnyhfxjd | 10.21.18.160:33738 | fxjd | Prepare     |   38680 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 554849 | hnyhfxjd | 10.21.18.160:33792 | fxjd | Prepare     |   38607 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 554881 | hnyhfxjd | 10.21.18.160:33960 | fxjd | Prepare     |   38423 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555142 | hnyhfxjd | 10.21.18.160:35304 | fxjd | Prepare     |   36832 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555163 | hnyhfxjd | 10.21.18.160:35406 | fxjd | Prepare     |   36721 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555184 | hnyhfxjd | 10.21.18.160:35512 | fxjd | Prepare     |   36620 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555304 | hnyhfxjd | 10.21.18.160:36126 | fxjd | Prepare     |   35872 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555326 | hnyhfxjd | 10.21.18.160:36232 | fxjd | Prepare     |   35806 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555347 | hnyhfxjd | 10.21.18.160:36340 | fxjd | Prepare     |   35633 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555368 | hnyhfxjd | 10.21.18.160:36444 | fxjd | Prepare     |   35547 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555389 | hnyhfxjd | 10.21.18.160:36550 | fxjd | Prepare     |   35392 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555429 | hnyhfxjd | 10.21.18.160:36754 | fxjd | Prepare     |   35176 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555430 | hnyhfxjd | 10.21.18.160:36756 | fxjd | Prepare     |   35190 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555452 | hnyhfxjd | 10.21.18.160:36864 | fxjd | Prepare     |   35063 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555473 | hnyhfxjd | 10.21.18.160:36966 | fxjd | Prepare     |   34951 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555493 | hnyhfxjd | 10.21.18.160:37070 | fxjd | Prepare     |   34806 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555503 | hnyhfxjd | 10.21.18.160:37122 | fxjd | Prepare     |   34747 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555505 | hnyhfxjd | 10.21.18.160:37126 | fxjd | Prepare     |   34757 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555526 | hnyhfxjd | 10.21.18.160:37230 | fxjd | Prepare     |   34624 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555537 | hnyhfxjd | 10.21.18.160:37280 | fxjd | Prepare     |   34570 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555547 | hnyhfxjd | 10.21.18.160:37334 | fxjd | Prepare     |   34506 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555548 | hnyhfxjd | 10.21.18.160:37336 | fxjd | Prepare     |   34541 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555590 | hnyhfxjd | 10.21.18.160:37542 | fxjd | Prepare     |   34258 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555602 | hnyhfxjd | 10.21.18.160:37598 | fxjd | Prepare     |   34219 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555610 | hnyhfxjd | 10.21.18.160:37644 | fxjd | Prepare     |   34133 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555611 | hnyhfxjd | 10.21.18.160:37646 | fxjd | Prepare     |   34139 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555613 | hnyhfxjd | 10.21.18.160:37650 | fxjd | Prepare     |   34142 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555642 | hnyhfxjd | 10.21.18.160:37800 | fxjd | Prepare     |   33955 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 555644 | hnyhfxjd | 10.21.18.160:37804 | fxjd | Prepare     |   33969 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555645 | hnyhfxjd | 10.21.18.160:37806 | fxjd | Prepare     |   33979 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 555659 | hnyhfxjd | 10.21.18.160:37868 | fxjd | Prepare     |   33915 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 555670 | hnyhfxjd | 10.21.18.160:37922 | fxjd | Prepare     |   33880 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555677 | hnyhfxjd | 10.21.18.160:37970 | fxjd | Prepare     |   33795 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555679 | hnyhfxjd | 10.21.18.160:37972 | fxjd | Prepare     |   33824 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555691 | hnyhfxjd | 10.21.18.160:38026 | fxjd | Prepare     |   33723 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555692 | hnyhfxjd | 10.21.18.160:38028 | fxjd | Prepare     |   33750 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555704 | hnyhfxjd | 10.21.18.160:38082 | fxjd | Prepare     |   33705 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555714 | hnyhfxjd | 10.21.18.160:38136 | fxjd | Prepare     |   33644 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555724 | hnyhfxjd | 10.21.18.160:38188 | fxjd | Prepare     |   33562 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555725 | hnyhfxjd | 10.21.18.160:38190 | fxjd | Prepare     |   33568 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 555736 | hnyhfxjd | 10.21.18.160:38242 | fxjd | Prepare     |   33474 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 555737 | hnyhfxjd | 10.21.18.160:38244 | fxjd | Prepare     |   33503 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555756 | hnyhfxjd | 10.21.18.160:38342 | fxjd | Prepare     |   33379 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555767 | hnyhfxjd | 10.21.18.160:38400 | fxjd | Prepare     |   33318 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555768 | hnyhfxjd | 10.21.18.160:38402 | fxjd | Prepare     |   33321 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555778 | hnyhfxjd | 10.21.18.160:38452 | fxjd | Prepare     |   33248 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555779 | hnyhfxjd | 10.21.18.160:38454 | fxjd | Prepare     |   33257 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555789 | hnyhfxjd | 10.21.18.160:38504 | fxjd | Prepare     |   33121 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555795 | hnyhfxjd | 10.21.18.160:38548 | fxjd | Prepare     |   33149 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 555805 | hnyhfxjd | 10.21.18.160:38600 | fxjd | Prepare     |   33002 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555807 | hnyhfxjd | 10.21.18.160:38604 | fxjd | Prepare     |   33006 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555808 | hnyhfxjd | 10.21.18.160:38606 | fxjd | Prepare     |   33107 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555814 | hnyhfxjd | 10.21.18.160:38648 | fxjd | Prepare     |   33038 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555824 | hnyhfxjd | 10.21.18.160:38700 | fxjd | Prepare     |   32951 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555826 | hnyhfxjd | 10.21.18.160:38704 | fxjd | Prepare     |   32953 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555836 | hnyhfxjd | 10.21.18.160:38752 | fxjd | Prepare     |   32919 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555855 | hnyhfxjd | 10.21.18.160:38854 | fxjd | Prepare     |   32757 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555857 | hnyhfxjd | 10.21.18.160:38858 | fxjd | Prepare     |   32809 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555869 | hnyhfxjd | 10.21.18.160:38914 | fxjd | Prepare     |   32703 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555880 | hnyhfxjd | 10.21.18.160:38966 | fxjd | Prepare     |   32633 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555899 | hnyhfxjd | 10.21.18.160:39066 | fxjd | Prepare     |   32529 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555901 | hnyhfxjd | 10.21.18.160:39070 | fxjd | Prepare     |   32562 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555912 | hnyhfxjd | 10.21.18.160:39124 | fxjd | Prepare     |   32463 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555913 | hnyhfxjd | 10.21.18.160:39126 | fxjd | Prepare     |   32484 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 555925 | hnyhfxjd | 10.21.18.160:39182 | fxjd | Prepare     |   32413 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 555966 | hnyhfxjd | 10.21.18.160:39388 | fxjd | Prepare     |   32195 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 555977 | hnyhfxjd | 10.21.18.160:39444 | fxjd | Prepare     |   32149 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556047 | hnyhfxjd | 10.21.18.160:39802 | fxjd | Prepare     |   31701 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556048 | hnyhfxjd | 10.21.18.160:39804 | fxjd | Prepare     |   31702 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556070 | hnyhfxjd | 10.21.18.160:39910 | fxjd | Prepare     |   31564 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556081 | hnyhfxjd | 10.21.18.160:39964 | fxjd | Prepare     |   31526 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556092 | hnyhfxjd | 10.21.18.160:40018 | fxjd | Prepare     |   31480 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556113 | hnyhfxjd | 10.21.18.160:40120 | fxjd | Prepare     |   31339 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556123 | hnyhfxjd | 10.21.18.160:40174 | fxjd | Prepare     |   31254 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556124 | hnyhfxjd | 10.21.18.160:40176 | fxjd | Prepare     |   31272 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556195 | hnyhfxjd | 10.21.18.160:40542 | fxjd | Prepare     |   30886 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556216 | hnyhfxjd | 10.21.18.160:40642 | fxjd | Prepare     |   30736 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556226 | hnyhfxjd | 10.21.18.160:40694 | fxjd | Prepare     |   30659 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556227 | hnyhfxjd | 10.21.18.160:40696 | fxjd | Prepare     |   30707 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556239 | hnyhfxjd | 10.21.18.160:40754 | fxjd | Prepare     |   30612 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556260 | hnyhfxjd | 10.21.18.160:40860 | fxjd | Prepare     |   30486 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556270 | hnyhfxjd | 10.21.18.160:40910 | fxjd | Prepare     |   30430 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556271 | hnyhfxjd | 10.21.18.160:40912 | fxjd | Prepare     |   30464 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556283 | hnyhfxjd | 10.21.18.160:40966 | fxjd | Prepare     |   30370 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556304 | hnyhfxjd | 10.21.18.160:41070 | fxjd | Prepare     |   30245 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556315 | hnyhfxjd | 10.21.18.160:41124 | fxjd | Prepare     |   30186 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556356 | hnyhfxjd | 10.21.18.160:41330 | fxjd | Prepare     |   29965 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556406 | hnyhfxjd | 10.21.18.160:41584 | fxjd | Prepare     |   29640 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556407 | hnyhfxjd | 10.21.18.160:41586 | fxjd | Prepare     |   29642 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556419 | hnyhfxjd | 10.21.18.160:41642 | fxjd | Prepare     |   29623 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556440 | hnyhfxjd | 10.21.18.160:41744 | fxjd | Prepare     |   29487 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556461 | hnyhfxjd | 10.21.18.160:41852 | fxjd | Prepare     |   29382 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556502 | hnyhfxjd | 10.21.18.160:42060 | fxjd | Prepare     |   29110 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556583 | hnyhfxjd | 10.21.18.160:42476 | fxjd | Prepare     |   28666 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556674 | hnyhfxjd | 10.21.18.160:42940 | fxjd | Prepare     |   28088 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556835 | hnyhfxjd | 10.21.18.160:43764 | fxjd | Prepare     |   27117 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 556846 | hnyhfxjd | 10.21.18.160:43818 | fxjd | Prepare     |   27077 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 556877 | hnyhfxjd | 10.21.18.160:43978 | fxjd | Prepare     |   26910 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556898 | hnyhfxjd | 10.21.18.160:44082 | fxjd | Prepare     |   26765 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 556939 | hnyhfxjd | 10.21.18.160:44290 | fxjd | Prepare     |   26566 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557250 | hnyhfxjd | 10.21.18.160:45882 | fxjd | Prepare     |   24656 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557271 | hnyhfxjd | 10.21.18.160:45988 | fxjd | Prepare     |   24569 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557292 | hnyhfxjd | 10.21.18.160:46090 | fxjd | Prepare     |   24440 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557303 | hnyhfxjd | 10.21.18.160:46142 | fxjd | Prepare     |   24366 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557355 | hnyhfxjd | 10.21.18.160:46404 | fxjd | Prepare     |   24059 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557444 | hnyhfxjd | 10.21.18.160:46864 | fxjd | Prepare     |   23537 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557472 | hnyhfxjd | 10.21.18.160:47014 | fxjd | Prepare     |   23364 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557473 | hnyhfxjd | 10.21.18.160:47016 | fxjd | Prepare     |   23379 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557476 | hnyhfxjd | 10.21.18.160:47022 | fxjd | Prepare     |   23380 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557489 | hnyhfxjd | 10.21.18.160:47078 | fxjd | Prepare     |   23304 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557519 | hnyhfxjd | 10.21.18.160:47230 | fxjd | Prepare     |   23105 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557530 | hnyhfxjd | 10.21.18.160:47286 | fxjd | Prepare     |   23084 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557539 | hnyhfxjd | 10.21.18.160:47334 | fxjd | Prepare     |   22982 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557551 | hnyhfxjd | 10.21.18.160:47388 | fxjd | Prepare     |   22947 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557582 | hnyhfxjd | 10.21.18.160:47546 | fxjd | Prepare     |   22730 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557594 | hnyhfxjd | 10.21.18.160:47602 | fxjd | Prepare     |   22705 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557614 | hnyhfxjd | 10.21.18.160:47702 | fxjd | Prepare     |   22590 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557635 | hnyhfxjd | 10.21.18.160:47810 | fxjd | Prepare     |   22453 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557647 | hnyhfxjd | 10.21.18.160:47864 | fxjd | Prepare     |   22387 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557707 | hnyhfxjd | 10.21.18.160:48168 | fxjd | Prepare     |   22040 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557738 | hnyhfxjd | 10.21.18.160:48324 | fxjd | Prepare     |   21866 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557759 | hnyhfxjd | 10.21.18.160:48428 | fxjd | Prepare     |   21725 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557800 | hnyhfxjd | 10.21.18.160:48638 | fxjd | Prepare     |   21486 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557821 | hnyhfxjd | 10.21.18.160:48742 | fxjd | Prepare     |   21385 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557842 | hnyhfxjd | 10.21.18.160:48848 | fxjd | Prepare     |   21265 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557854 | hnyhfxjd | 10.21.18.160:48906 | fxjd | Prepare     |   21222 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557874 | hnyhfxjd | 10.21.18.160:49008 | fxjd | Prepare     |   21072 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557886 | hnyhfxjd | 10.21.18.160:49066 | fxjd | Prepare     |   21030 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557917 | hnyhfxjd | 10.21.18.160:49220 | fxjd | Prepare     |   20825 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557928 | hnyhfxjd | 10.21.18.160:49272 | fxjd | Prepare     |   20775 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 557969 | hnyhfxjd | 10.21.18.160:49480 | fxjd | Prepare     |   20541 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558030 | hnyhfxjd | 10.21.18.160:49790 | fxjd | Prepare     |   20196 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558131 | hnyhfxjd | 10.21.18.160:50310 | fxjd | Prepare     |   19592 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558142 | hnyhfxjd | 10.21.18.160:50362 | fxjd | Prepare     |   19536 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558152 | hnyhfxjd | 10.21.18.160:50414 | fxjd | Prepare     |   19470 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558153 | hnyhfxjd | 10.21.18.160:50416 | fxjd | Prepare     |   19474 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558174 | hnyhfxjd | 10.21.18.160:50518 | fxjd | Prepare     |   19325 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558234 | hnyhfxjd | 10.21.18.160:50828 | fxjd | Prepare     |   18985 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558246 | hnyhfxjd | 10.21.18.160:50884 | fxjd | Prepare     |   18913 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558257 | hnyhfxjd | 10.21.18.160:50938 | fxjd | Prepare     |   18875 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558277 | hnyhfxjd | 10.21.18.160:51042 | fxjd | Prepare     |   18763 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558288 | hnyhfxjd | 10.21.18.160:51098 | fxjd | Prepare     |   18664 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558289 | hnyhfxjd | 10.21.18.160:51100 | fxjd | Prepare     |   18669 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558300 | hnyhfxjd | 10.21.18.160:51154 | fxjd | Prepare     |   18630 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558318 | hnyhfxjd | 10.21.18.160:51252 | fxjd | Prepare     |   18493 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558353 | hnyhfxjd | 10.21.18.160:51448 | fxjd | Prepare     |   18252 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558363 | hnyhfxjd | 10.21.18.160:51498 | fxjd | Prepare     |   18077 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558388 | hnyhfxjd | 10.21.18.160:51642 | fxjd | Prepare     |   18042 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558399 | hnyhfxjd | 10.21.18.160:51694 | fxjd | Prepare     |   17952 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558410 | hnyhfxjd | 10.21.18.160:51748 | fxjd | Prepare     |   17868 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558441 | hnyhfxjd | 10.21.18.160:51904 | fxjd | Prepare     |   17730 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558481 | hnyhfxjd | 10.21.18.160:52104 | fxjd | Prepare     |   17493 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558532 | hnyhfxjd | 10.21.18.160:52364 | fxjd | Prepare     |   17151 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558543 | hnyhfxjd | 10.21.18.160:52420 | fxjd | Prepare     |   17112 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558665 | hnyhfxjd | 10.21.18.160:53040 | fxjd | Prepare     |   16368 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558686 | hnyhfxjd | 10.21.18.160:53142 | fxjd | Prepare     |   16275 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558757 | hnyhfxjd | 10.21.18.160:53508 | fxjd | Prepare     |   15853 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558768 | hnyhfxjd | 10.21.18.160:53558 | fxjd | Prepare     |   15812 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558789 | hnyhfxjd | 10.21.18.160:53662 | fxjd | Prepare     |   15682 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 558920 | hnyhfxjd | 10.21.18.160:54334 | fxjd | Prepare     |   14888 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559011 | hnyhfxjd | 10.21.18.160:54798 | fxjd | Prepare     |   14339 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559042 | hnyhfxjd | 10.21.18.160:54954 | fxjd | Prepare     |   14201 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559063 | hnyhfxjd | 10.21.18.160:55058 | fxjd | Prepare     |   14065 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559074 | hnyhfxjd | 10.21.18.160:55110 | fxjd | Prepare     |   13991 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559095 | hnyhfxjd | 10.21.18.160:55214 | fxjd | Prepare     |   13884 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559115 | hnyhfxjd | 10.21.18.160:55318 | fxjd | Prepare     |   13751 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559116 | hnyhfxjd | 10.21.18.160:55320 | fxjd | Prepare     |   13756 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559158 | hnyhfxjd | 10.21.18.160:55528 | fxjd | Prepare     |   13494 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559179 | hnyhfxjd | 10.21.18.160:55632 | fxjd | Prepare     |   13412 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559201 | hnyhfxjd | 10.21.18.160:55740 | fxjd | Prepare     |   13290 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559301 | hnyhfxjd | 10.21.18.160:56250 | fxjd | Prepare     |   12692 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559340 | hnyhfxjd | 10.21.18.160:56452 | fxjd | Prepare     |   12445 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559342 | hnyhfxjd | 10.21.18.160:56456 | fxjd | Prepare     |   12460 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559414 | hnyhfxjd | 10.21.18.160:56828 | fxjd | Prepare     |   12016 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559425 | hnyhfxjd | 10.21.18.160:56878 | fxjd | Prepare     |   11984 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559466 | hnyhfxjd | 10.21.18.160:57086 | fxjd | Prepare     |   11710 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559577 | hnyhfxjd | 10.21.18.160:57654 | fxjd | Prepare     |   11041 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559637 | hnyhfxjd | 10.21.18.160:57960 | fxjd | Prepare     |   10671 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559638 | hnyhfxjd | 10.21.18.160:57962 | fxjd | Prepare     |   10704 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559840 | hnyhfxjd | 10.21.18.160:58988 | fxjd | Prepare     |    9525 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559850 | hnyhfxjd | 10.21.18.160:59038 | fxjd | Prepare     |    9408 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559851 | hnyhfxjd | 10.21.18.160:59040 | fxjd | Prepare     |    9427 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559893 | hnyhfxjd | 10.21.18.160:59252 | fxjd | Prepare     |    9198 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559914 | hnyhfxjd | 10.21.18.160:59352 | fxjd | Prepare     |    9105 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559945 | hnyhfxjd | 10.21.18.160:59510 | fxjd | Prepare     |    8916 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559956 | hnyhfxjd | 10.21.18.160:59562 | fxjd | Prepare     |    8848 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559976 | hnyhfxjd | 10.21.18.160:59666 | fxjd | Prepare     |    8697 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 559977 | hnyhfxjd | 10.21.18.160:59668 | fxjd | Prepare     |    8726 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 560089 | hnyhfxjd | 10.21.18.160:60234 | fxjd | Prepare     |    8044 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 560100 | hnyhfxjd | 10.21.18.160:60288 | fxjd | Prepare     |    7972 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 560121 | hnyhfxjd | 10.21.18.160:60394 | fxjd | Prepare     |    7891 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 560132 | hnyhfxjd | 10.21.18.160:60448 | fxjd | Prepare     |    7789 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 560153 | hnyhfxjd | 10.21.18.160:60552 | fxjd | Prepare     |    7720 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 560164 | hnyhfxjd | 10.21.18.160:60608 | fxjd | Prepare     |    7644 | Opening tables                                         32_0_, this_.BUSINUM as BUSINUM32_0_, this_.SYSID as SYSID32_0_, this_.DATAMAP  |
| 561061 | hnyhfxjd | 10.21.18.160:36862 | fxjd | Query       |    2546 | Opening tables                                         5_0_, this_.CODE as CODE5_0_, this_.NAME as NAME5_0_, this_.FILE_HTTP_IP as FIL |
| 561146 | hnyhfxjd | 10.21.18.161:53376 | fxjd | Prepare     |    2254 | Opening tables                                         _ from SM_USER this_ where this_.USER_CODE=?                                    |
| 561147 | hnyhfxjd | 10.19.22.18:52145  | NULL | Query       |    2228 | Opening tables                                         EFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME FROM information_schema.SCHEM |
| 561184 | igcam    | 10.19.22.18:52170  | NULL | Query       |    1661 | Opening tables                                         EFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME FROM information_schema.SCHEM |
| 561192 | hnyhfxjd | 10.21.18.160:37608 | fxjd | Query       |    1586 | Opening tables                                                                                                                         |
| 561303 | hnyhfxjd | 10.21.18.161:54710 | fxjd | Sleep       |     413 |                                                                                                                                        |
| 561304 | hnyhfxjd | 10.21.18.161:54712 | fxjd | Sleep       |     413 |                                                                                                                                        |
| 561306 | hnyhfxjd | 10.21.18.160:38454 | fxjd | Sleep       |     389 |                                                                                                                                        |
| 561307 | hnyhfxjd | 10.21.18.160:38456 | fxjd | Sleep       |     389 |                                                                                                                                        |
| 561308 | hnyhfxjd | 10.21.18.161:54750 | fxjd | Sleep       |     353 |                                                                                                                                        |
| 561309 | hnyhfxjd | 10.21.18.161:54752 | fxjd | Sleep       |     353 |                                                                                                                                        |
| 561310 | hnyhfxjd | 10.21.18.161:54754 | fxjd | Sleep       |     353 |                                                                                                                                        |
| 561311 | hnyhfxjd | 10.21.18.160:38492 | fxjd | Sleep       |     331 |                                                                                                                                        |
| 561312 | hnyhfxjd | 10.21.18.160:38494 | fxjd | Sleep       |     331 |                                                                                                                                        |
| 561313 | hnyhfxjd | 10.21.18.160:38496 | fxjd | Sleep       |     331 |                                                                                                                                        |
| 561314 | hnyhfxjd | 10.21.18.161:54792 | fxjd | Sleep       |     293 |                                                                                                                                        |
| 561315 | hnyhfxjd | 10.21.18.161:54794 | fxjd | Sleep       |     293 |                                                                                                                                        |
| 561316 | hnyhfxjd | 10.21.18.161:54796 | fxjd | Sleep       |     293 |                                                                                                                                        |
| 561317 | hnyhfxjd | 10.21.18.160:38536 | fxjd | Sleep       |     271 |                                                                                                                                        |
| 561318 | hnyhfxjd | 10.21.18.160:38538 | fxjd | Sleep       |     271 |                                                                                                                                        |
| 561319 | hnyhfxjd | 10.21.18.160:38540 | fxjd | Sleep       |     271 |                                                                                                                                        |
| 561320 | hnyhfxjd | 10.21.18.161:54836 | fxjd | Sleep       |     233 |                                                                                                                                        |
| 561321 | hnyhfxjd | 10.21.18.161:54838 | fxjd | Sleep       |     233 |                                                                                                                                        |
| 561322 | hnyhfxjd | 10.21.18.161:54840 | fxjd | Sleep       |     233 |                                                                                                                                        |
| 561323 | hnyhfxjd | 10.21.18.160:38578 | fxjd | Sleep       |     211 |                                                                                                                                        |
| 561324 | hnyhfxjd | 10.21.18.160:38580 | fxjd | Sleep       |     211 |                                                                                                                                        |
| 561325 | hnyhfxjd | 10.21.18.160:38582 | fxjd | Sleep       |     211 |                                                                                                                                        |
| 561326 | hnyhfxjd | 10.21.18.161:54876 | fxjd | Sleep       |     173 |                                                                                                                                        |
| 561327 | hnyhfxjd | 10.21.18.161:54878 | fxjd | Sleep       |     173 |                                                                                                                                        |
| 561328 | hnyhfxjd | 10.21.18.161:54880 | fxjd | Sleep       |     173 |                                                                                                                                        |
| 561329 | hnyhfxjd | 10.21.18.160:38622 | fxjd | Sleep       |     151 |                                                                                                                                        |
| 561330 | hnyhfxjd | 10.21.18.160:38624 | fxjd | Sleep       |     151 |                                                                                                                                        |
| 561331 | hnyhfxjd | 10.21.18.160:38626 | fxjd | Sleep       |     151 |                                                                                                                                        |
| 561332 | hnyhfxjd | 10.21.18.161:54924 | fxjd | Sleep       |     113 |                                                                                                                                        |
| 561333 | hnyhfxjd | 10.21.18.161:54926 | fxjd | Sleep       |     113 |                                                                                                                                        |
| 561334 | hnyhfxjd | 10.21.18.161:54928 | fxjd | Sleep       |     113 |                                                                                                                                        |
| 561335 | hnyhfxjd | 10.21.18.160:38666 | fxjd | Sleep       |      91 |                                                                                                                                        |
| 561336 | hnyhfxjd | 10.21.18.160:38668 | fxjd | Sleep       |      91 |                                                                                                                                        |
| 561337 | hnyhfxjd | 10.21.18.160:38670 | fxjd | Sleep       |      91 |                                                                                                                                        |
| 561338 | hnyhfxjd | 10.21.18.161:54964 | fxjd | Sleep       |      53 |                                                                                                                                        |
| 561339 | hnyhfxjd | 10.21.18.161:54966 | fxjd | Sleep       |      53 |                                                                                                                                        |
| 561340 | hnyhfxjd | 10.21.18.161:54968 | fxjd | Sleep       |      53 |                                                                                                                                        |
| 561341 | hnyhfxjd | 10.21.18.160:38708 | fxjd | Sleep       |      31 |                                                                                                                                        |
| 561342 | hnyhfxjd | 10.21.18.160:38710 | fxjd | Sleep       |      31 |                                                                                                                                        |
| 561343 | hnyhfxjd | 10.21.18.160:38712 | fxjd | Sleep       |      31 |                                                                                                                                        |
| 561344 | root     | localhost          | NULL | Query       |       0 | init                                                                                                                                   |
+--------+----------+--------------------+------+-------------+---------+----------------------------------------------------------------------------------------------------------------------------------------+
251 rows in set (0.00 sec)

``` 

processlist ID: 553852, 对应 Thread 74, 访问 data_lock_waits

# 测试环境复现的排查

[xbhndbtestsa_202408012_v1.log](/assets/01KJBZEE9JTH4RKQG664GFW88H/xbhndbtestsa_202408012_v1.log)

排除不重要的堆栈: 

```
grep -L 'Protocol_classic::get_command' * | xargs grep -L 'fil_aio_wait' | xargs grep -L 'wait_time_low' | xargs grep -L 'srv_master_sleep' | xargs grep -L 'buf_resize_thread' | xargs grep -L 'buf_flush_page_cleaner_thread' | xargs grep -L 'listen_for_connection_event' | xargs grep -L 'Scheduler_dynamic' | xargs grep -L 'buf_dump_thread' | xargs grep -L 'block_until_new_connection' | xargs grep -L 'srv_worker_thread'  | xargs grep -L 'compress_gtid_table' | xargs grep -L 'signal_hand'  | xargs grep -L 'wait_for_slave_connection'   | xargs cat
``` 

确定trx_sys的地址: this=this@entry=0x7fb7d832cdc8, 去除: 

```
grep -L 'Protocol_classic::get_command' * | xargs grep -L 'fil_aio_wait' | xargs grep -L 'wait_time_low' | xargs grep -L 'srv_master_sleep' | xargs grep -L 'buf_resize_thread' | xargs grep -L 'buf_flush_page_cleaner_thread' | xargs grep -L 'listen_for_connection_event' | xargs grep -L 'Scheduler_dynamic' | xargs grep -L 'buf_dump_thread' | xargs grep -L 'block_until_new_connection' | xargs grep -L 'srv_worker_thread'  | xargs grep -L 'compress_gtid_table' | xargs grep -L 'signal_hand'  | xargs grep -L 'wait_for_slave_connection'    | xargs grep -L 'this=this@entry=0x7fb7d832cdc8' | xargs cat
``` 

剩下的线程: 

```
└──> grep -L 'Protocol_classic::get_command' * | xargs grep -L 'fil_aio_wait' | xargs grep -L 'wait_time_low' | xargs grep -L 'srv_master_sleep' | xargs grep -L 'buf_resize_thread' | xargs grep -L 'buf_flush_page_cleaner_thread' | xargs grep -L 'listen_for_connection_event' | xargs grep -L 'Scheduler_dynamic' | xargs grep -L 'buf_dump_thread' | xargs grep -L 'block_until_new_connection' | xargs grep -L 'srv_worker_thread'  | xargs grep -L 'compress_gtid_table' | xargs grep -L 'signal_hand'  | xargs grep -L 'wait_for_slave_connection'    | xargs grep -L 'this=this@entry=0x7fb7d832cdc8' | xargs cat
Thread 20 (LWP 2957227):
#0  0x00007fb7ecc59700 in nanosleep () from /usr/lib64/libpthread.so.0
#1  0x0000000002381d05 in std::this_thread::sleep_for<long, std::ratio<1l, 1l> > (__rtime=...) at /usr/include/c++/7.3.0/thread:376
#2  sync_array_print_long_waits (waiter=waiter@entry=0x7fb53f7fcb60, sema=sema@entry=0x7fb53f7fcb68) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:973
#3  0x000000000236c68c in srv_error_monitor_thread () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:1834
#4  0x000000000227c8d7 in std::__invoke_impl<void, void (*&)()> (__f=<optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:60
#5  std::__invoke<void (*&)()> (__fn=<synthetic pointer>: <optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:95
#6  std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (__args=..., this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:467
#7  std::_Bind<void (*())()>::operator()<, void>() (this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:551
#8  Runnable::operator()<void (*)()> (f=@0x7fb7da6c25b8: 0x236c4c0 <srv_error_monitor_thread()>, this=0x7fb7da6c25c0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/os0thread-create.h:101
#9  std::__invoke_impl<void, Runnable, void (*)()> (__f=...) at /usr/include/c++/7.3.0/bits/invoke.h:60
#10 std::__invoke<Runnable, void (*)()> (__fn=...) at /usr/include/c++/7.3.0/bits/invoke.h:95
#11 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul> (this=0x7fb7da6c25b8) at /usr/include/c++/7.3.0/thread:234
#12 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator() (this=0x7fb7da6c25b8) at /usr/include/c++/7.3.0/thread:243
#13 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run (this=0x7fb7da6c25b0) at /usr/include/c++/7.3.0/thread:186
#14 0x00007fb7ec75e95a in ?? () from /usr/lib64/libstdc++.so.6
#15 0x00007fb7ecc4ff2b in ?? () from /usr/lib64/libpthread.so.0
#16 0x00007fb7ec42634f in clone () from /usr/lib64/libc.so.6
Thread 46 (LWP 2966196):
#0  0x00007fb7ecc55dd2 in pthread_cond_timedwait () from /usr/lib64/libpthread.so.0
#1  0x00007fb7dcaeea12 in native_cond_timedwait (abstime=0x7fb7dc206500, mutex=0x7fb7dc206530, cond=0x7fb7dc206560) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/include/thr_cond.h:100
#2  my_cond_timedwait (abstime=0x7fb7dc206500, mp=0x7fb7dc206530, cond=0x7fb7dc206560) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/include/thr_cond.h:149
#3  inline_mysql_cond_timedwait (src_file=0x7fb7dcaf0588 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/plugin/connection_control/connection_delay.cc", src_line=481, abstime=0x7fb7dc206500, mutex=0x7fb7dc206530, that=0x7fb7dc206560) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/include/mysql/psi/mysql_cond.h:241
#4  connection_control::Connection_delay_action::conditional_wait (this=this@entry=0x6a29c80, thd=thd@entry=0x7fb4d40186f0, wait_time=wait_time@entry=100000) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/plugin/connection_control/connection_delay.cc:480
#5  0x00007fb7dcaeec6a in connection_control::Connection_delay_action::notify_event (this=0x6a29c80, thd=0x7fb4d40186f0, coordinator=0x6b34070, connection_event=0x7fb7dc206800, error_handler=0x7fb7dc206748) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/plugin/connection_control/connection_delay.cc:564
#6  0x00007fb7dcaec878 in connection_control::Connection_event_coordinator::notify_event (this=0x6b34070, thd=0x7fb4d40186f0, error_handler=error_handler@entry=0x7fb7dc206748, connection_event=0x7fb7dc206800) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/plugin/connection_control/connection_control_coordinator.cc:162
#7  0x00007fb7dcaece40 in connection_control_notify (thd=<optimized out>, event_class=<optimized out>, event=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/plugin/connection_control/connection_control.cc:131
#8  0x000000000102c5c1 in plugins_dispatch (arg=0x7fb7dc206760, plugin=<optimized out>, thd=0x7fb4d40186f0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_audit.cc:1348
#9  event_class_dispatch (thd=thd@entry=0x7fb4d40186f0, event_class=<optimized out>, event=event@entry=0x7fb7dc206800) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_audit.cc:1386
#10 0x000000000102c686 in event_class_dispatch_error (thd=thd@entry=0x7fb4d40186f0, event_class=event_class@entry=MYSQL_AUDIT_CONNECTION_CLASS, event_name=event_name@entry=0x3043cb0 "MYSQL_AUDIT_CONNECTION_CONNECT", event=event@entry=0x7fb7dc206800) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_audit.cc:1399
#11 0x000000000102d244 in mysql_audit_notify (thd=0x7fb4d40186f0, subclass=MYSQL_AUDIT_CONNECTION_CONNECT, subclass_name=0x3043cb0 "MYSQL_AUDIT_CONNECTION_CONNECT", errcode=1045) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_audit.cc:454
#12 0x000000000102d352 in mysql_audit_notify (thd=thd@entry=0x7fb4d40186f0, subclass=subclass@entry=MYSQL_AUDIT_CONNECTION_CONNECT, subclass_name=subclass_name@entry=0x3043cb0 "MYSQL_AUDIT_CONNECTION_CONNECT") at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_audit.cc:461
#13 0x0000000000ea0a32 in check_connection (thd=thd@entry=0x7fb4d40186f0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_connect.cc:653
#14 0x0000000000ea1d36 in login_connection (thd=0x7fb4d40186f0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_connect.cc:716
#15 thd_prepare_connection (thd=thd@entry=0x7fb4d40186f0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_connect.cc:889
#16 0x000000000101d646 in handle_connection (arg=arg@entry=0x67547d0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/conn_handler/connection_handler_per_thread.cc:297
#17 0x000000000262b551 in pfs_spawn_thread (arg=0x6afda30) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs.cc:2898
#18 0x00007fb7ecc4ff2b in ?? () from /usr/lib64/libpthread.so.0
#19 0x00007fb7ec42634f in clone () from /usr/lib64/libc.so.6
┌────[tachikoma@miao]───────[16:22:28]───────[~/Downloads/帐户资料系统/threads_xbhndbtestsa_202408012_v1]─────────────────────────────────────────────────────────────────────────
└──>
``` 

Thread 20: srv_error_monitor_thread, 触发了打印日志的动作, 并等待30s

Thread 46: Connection Control 插件, 进行等待

那么重要的只剩下trx_sys的线程: 

```
└──> grep -l 'this=this@entry=0x7fb7d832cdc8' * | xargs cat
Thread 21 (LWP 2957228):
#0  0x00007fb7ecc55a1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fb7d8329c50) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fb7d8329c50, reset_sig_count=3) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fb7d800b660, cell=@0x7fb5a652a920: 0x7fb7df05c130) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fb7d832cdc8, filename=filename@entry=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=line@entry=697, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fb7d832cdc8, max_spins=60, max_delay=6, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=697) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000022df6be in TTASEventMutex<GenericPolicy>::enter (line=697, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", max_delay=6, max_spins=30, this=0x7fb7d832cdc8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=697, name=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", n_delay=6, n_spins=30, this=0x7fb7d832cdc8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=0x7fb7d832cdc8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 MVCC::size (this=0x7fb7d8329eb0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc:697
#11 0x000000000236a7fc in srv_printf_innodb_monitor (file=<optimized out>, nowait=<optimized out>, trx_start_pos=trx_start_pos@entry=0x0, trx_end=trx_end@entry=0x0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:1440
#12 0x000000000236bed9 in srv_monitor_thread () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:1750
#13 0x000000000227c8d7 in std::__invoke_impl<void, void (*&)()> (__f=<optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:60
#14 std::__invoke<void (*&)()> (__fn=<synthetic pointer>: <optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:95
#15 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (__args=..., this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:467
#16 std::_Bind<void (*())()>::operator()<, void>() (this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:551
#17 Runnable::operator()<void (*)()> (f=@0x7fb7da6c2678: 0x236be20 <srv_monitor_thread()>, this=0x7fb7da6c2680) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/os0thread-create.h:101
#18 std::__invoke_impl<void, Runnable, void (*)()> (__f=...) at /usr/include/c++/7.3.0/bits/invoke.h:60
#19 std::__invoke<Runnable, void (*)()> (__fn=...) at /usr/include/c++/7.3.0/bits/invoke.h:95
#20 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul> (this=0x7fb7da6c2678) at /usr/include/c++/7.3.0/thread:234
#21 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator() (this=0x7fb7da6c2678) at /usr/include/c++/7.3.0/thread:243
#22 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run (this=0x7fb7da6c2670) at /usr/include/c++/7.3.0/thread:186
#23 0x00007fb7ec75e95a in ?? () from /usr/lib64/libstdc++.so.6
#24 0x00007fb7ecc4ff2b in ?? () from /usr/lib64/libpthread.so.0
#25 0x00007fb7ec42634f in clone () from /usr/lib64/libc.so.6
Thread 32 (LWP 2957242):
#0  0x00007fb7ecc55a1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fb7d8329c50) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fb7d8329c50, reset_sig_count=3) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fb7d800b660, cell=@0x7fb51fffea60: 0x7fb7df05c0f0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fb7d832cdc8, filename=filename@entry=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=line@entry=671, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fb7d832cdc8, max_spins=60, max_delay=6, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=671) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000022dfa9e in TTASEventMutex<GenericPolicy>::enter (line=671, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", max_delay=<optimized out>, max_spins=30, this=0x7fb7d832cdc8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=671, name=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", n_delay=<optimized out>, n_spins=30, this=0x7fb7d832cdc8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=0x7fb7d832cdc8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 MVCC::clone_oldest_view (this=0x7fb7d8329eb0, view=0x7fb7da678cd0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc:671
#11 0x0000000002399314 in trx_purge (n_purge_threads=4, batch_size=300, truncate=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0purge.cc:2409
#12 0x000000000236f196 in srv_do_purge (n_total_purged=<synthetic pointer>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:2923
#13 srv_purge_coordinator_thread () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:3101
#14 0x000000000227c8d7 in std::__invoke_impl<void, void (*&)()> (__f=<optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:60
#15 std::__invoke<void (*&)()> (__fn=<synthetic pointer>: <optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:95
#16 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (__args=..., this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:467
#17 std::_Bind<void (*())()>::operator()<, void>() (this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:551
#18 Runnable::operator()<void (*)()> (f=@0x6a479e8: 0x236ece0 <srv_purge_coordinator_thread()>, this=0x6a479f0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/os0thread-create.h:101
#19 std::__invoke_impl<void, Runnable, void (*)()> (__f=...) at /usr/include/c++/7.3.0/bits/invoke.h:60
#20 std::__invoke<Runnable, void (*)()> (__fn=...) at /usr/include/c++/7.3.0/bits/invoke.h:95
#21 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul> (this=0x6a479e8) at /usr/include/c++/7.3.0/thread:234
#22 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator() (this=0x6a479e8) at /usr/include/c++/7.3.0/thread:243
#23 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run (this=0x6a479e0) at /usr/include/c++/7.3.0/thread:186
#24 0x00007fb7ec75e95a in ?? () from /usr/lib64/libstdc++.so.6
#25 0x00007fb7ecc4ff2b in ?? () from /usr/lib64/libpthread.so.0
#26 0x00007fb7ec42634f in clone () from /usr/lib64/libc.so.6
Thread 40 (LWP 2957356):
#0  0x00007fb7ecc55a1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fb7d8329c50) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fb7d8329c50, reset_sig_count=3) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fb7d800b660, cell=@0x7fb7dc3b35e0: 0x7fb7df05c0b0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fb7d832cdc8, filename=filename@entry=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=line@entry=556, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fb7d832cdc8, max_spins=60, max_delay=6, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=556) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000022df31e in TTASEventMutex<GenericPolicy>::enter (line=556, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", max_delay=<optimized out>, max_spins=<optimized out>, this=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=556, name=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", n_delay=<optimized out>, n_spins=<optimized out>, this=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 MVCC::view_open (this=0x7fb7d8329eb0, view=@0x7fb7dec5c380: 0x7fb7d8354b20, trx=trx@entry=0x7fb7dec5c328) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc:556
#11 0x00000000023bd815 in trx_assign_read_view (trx=trx@entry=0x7fb7dec5c328) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0trx.cc:2264
#12 0x0000000002349097 in row_search_mvcc (buf=buf@entry=0x7fb4f419b008 "\377", mode=<optimized out>, mode@entry=PAGE_CUR_G, prebuilt=0x7fb4f4019c18, match_mode=match_mode@entry=0, direction=direction@entry=0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/row/row0sel.cc:4824
#13 0x00000000021e9315 in ha_innobase::index_read (this=0x7fb4f4186cf8, buf=0x7fb4f419b008 "\377", key_ptr=<optimized out>, key_len=<optimized out>, find_flag=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/ha_innodb.cc:9972
#14 0x00000000021c3a40 in ha_innobase::index_first (this=0x7fb4f4186cf8, buf=0x7fb4f419b008 "\377") at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/ha_innodb.cc:10322
#15 0x00000000021e9a8f in ha_innobase::rnd_next (this=0x7fb4f4186cf8, buf=0x7fb4f419b008 "\377") at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/ha_innodb.cc:10508
#16 0x00000000011305f3 in handler::ha_rnd_next (this=0x7fb4f4186cf8, buf=0x7fb4f419b008 "\377") at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/handler.cc:2985
#17 0x0000000000e305dd in TableScanIterator::Read (this=0x7fb4f4485d10) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/records.cc:361
#18 0x00000000013972c4 in FilterIterator::Read (this=0x7fb4f4485d40) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/composite_iterators.cc:72
#19 0x0000000001119f95 in read_all_rows (longest_addons=<synthetic pointer>, longest_key=<synthetic pointer>, found_rows=0x7fb7dc3b4be8, source_iterator=0x7fb4f4485d40, pq=0x0, tempfile=0x7fb7dc3b4930, chunk_file=0x7fb7dc3b4a60, fs_info=0x7fb4f4485d88, tables_to_get_rowid_for=0, tables=..., param=0x7fb4f4485788, thd=0x7fb4f40c14a0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/filesort.cc:988
#20 filesort (thd=0x7fb4f40c14a0, filesort=0x7fb4f4485740, source_iterator=0x7fb4f4485d40, tables_to_get_rowid_for=0, num_rows_estimate=300, fs_info=fs_info@entry=0x7fb4f4485d88, sort_result=0x7fb4f4485e10, found_rows=0x7fb7dc3b4be8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/filesort.cc:516
#21 0x0000000000e4137f in SortingIterator::DoSort (this=this@entry=0x7fb4f4485d60) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sorting_iterator.cc:533
#22 0x0000000000e413f1 in SortingIterator::Init (this=0x7fb4f4485d60) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sorting_iterator.cc:444
#23 0x0000000001397a46 in LimitOffsetIterator::Init (this=0x7fb4f4485eb0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/composite_iterators.cc:96
#24 0x0000000000fb492d in Query_expression::ExecuteIteratorQuery (this=0x7fb4f4480928, thd=thd@entry=0x7fb4f40c14a0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1224
#25 0x0000000000fb4bac in Query_expression::execute (this=<optimized out>, thd=0x7fb4f40c14a0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1284
#26 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fb4f44833b0, thd=0x7fb4f40c14a0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#27 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fb4f40c14a0, first_level=first_level@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#28 0x0000000000ef6045 in dispatch_sql_command (thd=thd@entry=0x7fb4f40c14a0, parser_state=parser_state@entry=0x7fb7dc3b6640) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:5033
#29 0x0000000000ef79b5 in dispatch_command (thd=thd@entry=0x7fb4f40c14a0, com_data=com_data@entry=0x7fb7dc3b6d30, command=COM_QUERY) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1863
#30 0x0000000000ef921c in do_command (thd=thd@entry=0x7fb4f40c14a0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1342
#31 0x000000000101d6f0 in handle_connection (arg=arg@entry=0x6c81f90) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/conn_handler/connection_handler_per_thread.cc:301
#32 0x000000000262b551 in pfs_spawn_thread (arg=0x6a849a0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs.cc:2898
#33 0x00007fb7ecc4ff2b in ?? () from /usr/lib64/libpthread.so.0
#34 0x00007fb7ec42634f in clone () from /usr/lib64/libc.so.6
Thread 45 (LWP 2963257):
#0  0x00007fb7ecc55a1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fb7d8329c50) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fb7d8329c50, reset_sig_count=3) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fb7d800b660, cell=@0x7fb7dc248220: 0x7fb7df05c030) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fb7d832cdc8, filename=filename@entry=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", line=line@entry=213, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=this@entry=0x7fb7d832cdc8, max_spins=60, max_spins@entry=30, max_delay=max_delay@entry=6, filename=filename@entry=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", line=line@entry=213) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000021de567 in TTASEventMutex<GenericPolicy>::enter (line=213, filename=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", max_delay=6, max_spins=30, this=0x7fb7d832cdc8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (this=0x7fb7d832cdc8, n_spins=30, n_delay=6, name=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", line=213) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  0x0000000002256996 in mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 trx_rw_min_trx_id () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic:213
#11 lock_sec_rec_read_check_and_lock (duration=duration@entry=lock_duration_t::REGULAR, block=block@entry=0x7fb5c7af4570, rec=rec@entry=0x7fb5ca8e2967 "0", index=index@entry=0x7fb4e0aa4af8, offsets=0x7fb7dc248b10, sel_mode=sel_mode@entry=SELECT_ORDINARY, mode=LOCK_S, gap_mode=0, thr=0x7fb4e0aa81b0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/lock/lock0lock.cc:5535
#12 0x00000000023447f5 in sel_set_rec_lock (pcur=pcur@entry=0x7fb4e0aa6e70, rec=rec@entry=0x7fb5ca8e2967 "0", index=index@entry=0x7fb4e0aa4af8, offsets=offsets@entry=0x7fb7dc248b10, sel_mode=SELECT_ORDINARY, mode=<optimized out>, type=0, thr=0x7fb4e0aa81b0, mtr=0x7fb7dc249150) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/row/row0sel.cc:1170
#13 0x0000000002349606 in row_search_mvcc (buf=buf@entry=0x7fb4e0a9ec98 "", mode=PAGE_CUR_GE, mode@entry=PAGE_CUR_UNSUPP, prebuilt=0x7fb4e0aa6c48, match_mode=match_mode@entry=1, direction=direction@entry=1) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/row/row0sel.cc:5225
#14 0x00000000021e9830 in ha_innobase::general_fetch (this=0x7fb4e0a9d5d8, buf=0x7fb4e0a9ec98 "", direction=1, match_mode=1) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/ha_innodb.cc:10224
#15 0x0000000001131b5c in handler::ha_index_next_same (this=0x7fb4e0a9d5d8, buf=0x7fb4e0a9ec98 "", key=0x7fb4e0a756b0 "", keylen=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/handler.cc:3458
#16 0x0000000000eb6c0b in RefIterator<false>::Read (this=0x7fb4e0aaa120) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_executor.cc:3795
#17 0x0000000000fb4a6b in Query_expression::ExecuteIteratorQuery (this=0x7fb4e09be990, thd=thd@entry=0x7fb4e0011f20) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1231
#18 0x0000000000fb4bac in Query_expression::execute (this=<optimized out>, thd=0x7fb4e0011f20) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1284
#19 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fb4e09c1778, thd=0x7fb4e0011f20) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#20 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fb4e0011f20, first_level=first_level@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#21 0x0000000000e4b631 in sp_instr_stmt::exec_core (this=0x7fb4e09c18f8, thd=0x7fb4e0011f20, nextp=0x7fb7dc24aafc) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:966
#22 0x0000000000e4d699 in sp_lex_instr::reset_lex_and_exec_core (this=this@entry=0x7fb4e09c18f8, thd=thd@entry=0x7fb4e0011f20, nextp=nextp@entry=0x7fb7dc24aafc, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:441
#23 0x0000000000e4e6a4 in sp_lex_instr::validate_lex_and_execute_core (this=this@entry=0x7fb4e09c18f8, thd=thd@entry=0x7fb4e0011f20, nextp=nextp@entry=0x7fb7dc24aafc, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:727
#24 0x0000000000e4ff4c in sp_instr_stmt::execute (this=0x7fb4e09c18f8, thd=0x7fb4e0011f20, nextp=0x7fb7dc24aafc) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:892
#25 0x0000000000e44166 in sp_head::execute (this=this@entry=0x7fb4e09998b8, thd=thd@entry=0x7fb4e0011f20, merge_da_on_success=merge_da_on_success@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2222
#26 0x0000000000e473f3 in sp_head::execute_procedure (this=this@entry=0x7fb4e09998b8, thd=thd@entry=0x7fb4e0011f20, args=0x7fb4e013af00) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2871
#27 0x00000000013521e6 in Sql_cmd_call::execute_inner (this=0x7fb4e013b648, thd=0x7fb4e0011f20) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_call.cc:235
#28 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fb4e013b648, thd=0x7fb4e0011f20) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#29 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fb4e0011f20, first_level=first_level@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#30 0x0000000000e4b631 in sp_instr_stmt::exec_core (this=0x7fb4e013b6c0, thd=0x7fb4e0011f20, nextp=0x7fb7dc24c59c) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:966
#31 0x0000000000e4d699 in sp_lex_instr::reset_lex_and_exec_core (this=this@entry=0x7fb4e013b6c0, thd=thd@entry=0x7fb4e0011f20, nextp=nextp@entry=0x7fb7dc24c59c, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:441
#32 0x0000000000e4e6a4 in sp_lex_instr::validate_lex_and_execute_core (this=this@entry=0x7fb4e013b6c0, thd=thd@entry=0x7fb4e0011f20, nextp=nextp@entry=0x7fb7dc24c59c, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:727
#33 0x0000000000e4ff4c in sp_instr_stmt::execute (this=0x7fb4e013b6c0, thd=0x7fb4e0011f20, nextp=0x7fb7dc24c59c) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:892
#34 0x0000000000e44166 in sp_head::execute (this=this@entry=0x7fb4e011ba78, thd=thd@entry=0x7fb4e0011f20, merge_da_on_success=merge_da_on_success@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2222
#35 0x0000000000e473f3 in sp_head::execute_procedure (this=this@entry=0x7fb4e011ba78, thd=thd@entry=0x7fb4e0011f20, args=0x7fb4e0037648) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2871
#36 0x00000000013521e6 in Sql_cmd_call::execute_inner (this=0x7fb4e0037b10, thd=0x7fb4e0011f20) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_call.cc:235
#37 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fb4e0037b10, thd=0x7fb4e0011f20) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#38 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fb4e0011f20, first_level=first_level@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#39 0x0000000000ef6045 in dispatch_sql_command (thd=thd@entry=0x7fb4e0011f20, parser_state=parser_state@entry=0x7fb7dc24e640) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:5033
#40 0x0000000000ef79b5 in dispatch_command (thd=thd@entry=0x7fb4e0011f20, com_data=com_data@entry=0x7fb7dc24ed30, command=COM_QUERY) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1863
#41 0x0000000000ef921c in do_command (thd=thd@entry=0x7fb4e0011f20) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1342
#42 0x000000000101d6f0 in handle_connection (arg=arg@entry=0x67cbdd0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/conn_handler/connection_handler_per_thread.cc:301
#43 0x000000000262b551 in pfs_spawn_thread (arg=0x6ac6480) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs.cc:2898
#44 0x00007fb7ecc4ff2b in ?? () from /usr/lib64/libpthread.so.0
#45 0x00007fb7ec42634f in clone () from /usr/lib64/libc.so.6
Thread 47 (LWP 2966486):
#0  0x00007fb7ecc55a1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fb7d8329c50) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fb7d8329c50, reset_sig_count=3) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fb7d800b660, cell=@0x7fb7dc1bca70: 0x7fb7df05c070) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fb7d832cdc8, filename=filename@entry=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", line=line@entry=856, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fb7d832cdc8, max_spins=60, max_delay=6, filename=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", line=856) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000022303d1 in TTASEventMutex<GenericPolicy>::enter (line=856, filename=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", max_delay=6, max_spins=30, this=0x7fb7d832cdc8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=856, name=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", n_delay=6, n_spins=30, this=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 Innodb_data_lock_wait_iterator::scan (this=0x7fb494595350, container=0x7fb4c80e86d0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc:856
#11 0x000000000265f72e in table_data_lock_waits::rnd_next (this=0x7fb4c80e8580) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/table_data_lock_waits.cc:185
#12 0x0000000002624cae in ha_perfschema::rnd_next (this=0x7fb4c8042938, buf=0x7fb4c8043fc8 "\377") at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/ha_perfschema.cc:1678
#13 0x00000000011306fc in handler::ha_rnd_next (this=0x7fb4c8042938, buf=0x7fb4c8043fc8 "\377") at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/handler.cc:2985
#14 0x0000000000e305dd in TableScanIterator::Read (this=0x7fb4c80c6040) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/records.cc:361
#15 0x0000000000fb4a6b in Query_expression::ExecuteIteratorQuery (this=0x7fb4c80c24d8, thd=thd@entry=0x7fb4c800e250) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1231
#16 0x0000000000fb4bac in Query_expression::execute (this=<optimized out>, thd=0x7fb4c800e250) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1284
#17 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fb4c80c4518, thd=0x7fb4c800e250) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#18 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fb4c800e250, first_level=first_level@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#19 0x0000000000ef6045 in dispatch_sql_command (thd=thd@entry=0x7fb4c800e250, parser_state=parser_state@entry=0x7fb7dc1be640) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:5033
#20 0x0000000000ef79b5 in dispatch_command (thd=thd@entry=0x7fb4c800e250, com_data=com_data@entry=0x7fb7dc1bed30, command=COM_QUERY) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1863
#21 0x0000000000ef921c in do_command (thd=thd@entry=0x7fb4c800e250) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1342
#22 0x000000000101d6f0 in handle_connection (arg=arg@entry=0x67843d0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/conn_handler/connection_handler_per_thread.cc:301
#23 0x000000000262b551 in pfs_spawn_thread (arg=0x69199b0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs.cc:2898
#24 0x00007fb7ecc4ff2b in ?? () from /usr/lib64/libpthread.so.0
#25 0x00007fb7ec42634f in clone () from /usr/lib64/libc.so.6
┌────[tachikoma@miao]───────[16:52:09]───────[~/Downloads/帐户资料系统/threads_xbhndbtestsa_202408012_v1]─────────────────────────────────────────────────────────────────────────
└──>
``` 

一共五个线程: 

  - Thread 21: srv_printf_innodb_monitor, 停在MVCC.size
  - Thread 32: srv_purge_coordinator_thread, trx_purge, 停在MVCC.clone_oldest_view
  - Thread 40: SQL处理, trx_assign_read_view, 停在MVCC::view_open
  - Thread 45: CALL SQL处理, 停在trx_rw_min_trx_id
  - Thread 47: Innodb_data_lock_wait_iterator, 

# 生产环境复现的排查

[20240802statc.log](/assets/01KJBZEE9JTH4RKQG664GFW88H/20240802statc.log)

TODO

# 突然的想法

排查在error log的锁列表中 不存在, 但线程中确实 等待锁 的线程, 这些线程可能不停占用锁形成了活锁

error log中的锁列表:

```
(Thread 56)
--Thread 140581528016640 has waited at trx0sys.ic line 213 for 227 seconds the semaphore:
Mutex at 0x7fddfc32c958, Mutex TRX_SYS created trx0sys.cc:565, lock var 1

(trx_disconnect_from_mysql, 一共有10个, 这里只是一个)
--Thread 140581530375936 has waited at trx0trx.cc line 621 for 133 seconds the semaphore:
Mutex at 0x7fddfc32c958, Mutex TRX_SYS created trx0sys.cc:565, lock var 1

(Thread 74)
--Thread 140579655780096 has waited at p_s.cc line 856 for 226 seconds the semaphore:
Mutex at 0x7fddfc32c958, Mutex TRX_SYS created trx0sys.cc:565, lock var 1

(Thread 32)
--Thread 140579715413760 has waited at read0read.cc line 671 for 249 seconds the semaphore:
Mutex at 0x7fddfc32c958, Mutex TRX_SYS created trx0sys.cc:565, lock var 1

(trx_allocate_for_mysql, 一共有207个, 这里只是一个)
--Thread 140579655190272 has waited at trx0trx.cc line 518 for 98 seconds the semaphore:
Mutex at 0x7fddfc32c958, Mutex TRX_SYS created trx0sys.cc:565, lock var 1

``` 

筛选线程表: 

```
└──> grep -l '0x7fddfc32c958' * | xargs grep -L 'open_table_from_share' | xargs grep -L 'ha_close_connection' | xargs grep -L 'open_tables_for_query' | xargs cat
Thread 21 (LWP 3750803):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc329910) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc329910, reset_sig_count=2) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdb64716920: 0x7fde02846170) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fddfc32c958, filename=filename@entry=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=line@entry=697, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fddfc32c958, max_spins=60, max_delay=6, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=697) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000022df6be in TTASEventMutex<GenericPolicy>::enter (line=697, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", max_delay=6, max_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=697, name=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", n_delay=6, n_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 MVCC::size (this=0x7fddfc329b90) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc:697
#11 0x000000000236a7fc in srv_printf_innodb_monitor (file=<optimized out>, nowait=<optimized out>, trx_start_pos=trx_start_pos@entry=0x0, trx_end=trx_end@entry=0x0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:1440
#12 0x000000000236bed9 in srv_monitor_thread () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:1750
#13 0x000000000227c8d7 in std::__invoke_impl<void, void (*&)()> (__f=<optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:60
#14 std::__invoke<void (*&)()> (__fn=<synthetic pointer>: <optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:95
#15 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (__args=..., this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:467
#16 std::_Bind<void (*())()>::operator()<, void>() (this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:551
#17 Runnable::operator()<void (*)()> (f=@0x7fddfe6bdcc8: 0x236be20 <srv_monitor_thread()>, this=0x7fddfe6bdcd0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/os0thread-create.h:101
#18 std::__invoke_impl<void, Runnable, void (*)()> (__f=...) at /usr/include/c++/7.3.0/bits/invoke.h:60
#19 std::__invoke<Runnable, void (*)()> (__fn=...) at /usr/include/c++/7.3.0/bits/invoke.h:95
#20 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul> (this=0x7fddfe6bdcc8) at /usr/include/c++/7.3.0/thread:234
#21 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator() (this=0x7fddfe6bdcc8) at /usr/include/c++/7.3.0/thread:243
#22 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run (this=0x7fddfe6bdcc0) at /usr/include/c++/7.3.0/thread:186
#23 0x00007fde0ff4895a in ?? () from /usr/lib64/libstdc++.so.6
#24 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#25 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
Thread 32 (LWP 3750817):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc329910) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc329910, reset_sig_count=2) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdb43ffea60: 0x7fde02846130) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fddfc32c958, filename=filename@entry=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=line@entry=671, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fddfc32c958, max_spins=60, max_delay=6, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", line=671) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000022dfa9e in TTASEventMutex<GenericPolicy>::enter (line=671, filename=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", max_delay=<optimized out>, max_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=671, name=0x3099a90 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc", n_delay=<optimized out>, n_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 MVCC::clone_oldest_view (this=0x7fddfc329b90, view=0x7fddfe6780a0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/read/read0read.cc:671
#11 0x0000000002399314 in trx_purge (n_purge_threads=4, batch_size=300, truncate=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/trx/trx0purge.cc:2409
#12 0x000000000236f196 in srv_do_purge (n_total_purged=<synthetic pointer>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:2923
#13 srv_purge_coordinator_thread () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/srv/srv0srv.cc:3101
#14 0x000000000227c8d7 in std::__invoke_impl<void, void (*&)()> (__f=<optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:60
#15 std::__invoke<void (*&)()> (__fn=<synthetic pointer>: <optimized out>) at /usr/include/c++/7.3.0/bits/invoke.h:95
#16 std::_Bind<void (*())()>::__call<void>(std::tuple<>&&, std::_Index_tuple<>) (__args=..., this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:467
#17 std::_Bind<void (*())()>::operator()<, void>() (this=<synthetic pointer>) at /usr/include/c++/7.3.0/functional:551
#18 Runnable::operator()<void (*)()> (f=@0x6b6df78: 0x236ece0 <srv_purge_coordinator_thread()>, this=0x6b6df80) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/os0thread-create.h:101
#19 std::__invoke_impl<void, Runnable, void (*)()> (__f=...) at /usr/include/c++/7.3.0/bits/invoke.h:60
#20 std::__invoke<Runnable, void (*)()> (__fn=...) at /usr/include/c++/7.3.0/bits/invoke.h:95
#21 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::_M_invoke<0ul, 1ul> (this=0x6b6df78) at /usr/include/c++/7.3.0/thread:234
#22 std::thread::_Invoker<std::tuple<Runnable, void (*)()> >::operator() (this=0x6b6df78) at /usr/include/c++/7.3.0/thread:243
#23 std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)()> > >::_M_run (this=0x6b6df70) at /usr/include/c++/7.3.0/thread:186
#24 0x00007fde0ff4895a in ?? () from /usr/lib64/libstdc++.so.6
#25 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#26 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
Thread 56 (LWP 2783865):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc329910) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc329910, reset_sig_count=2) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdbb009a220: 0x7fde02846070) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fddfc32c958, filename=filename@entry=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", line=line@entry=213, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=this@entry=0x7fddfc32c958, max_spins=60, max_spins@entry=30, max_delay=max_delay@entry=6, filename=filename@entry=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", line=line@entry=213) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000021de567 in TTASEventMutex<GenericPolicy>::enter (line=213, filename=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", max_delay=6, max_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (this=0x7fddfc32c958, n_spins=30, n_delay=6, name=0x3094548 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic", line=213) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  0x0000000002256996 in mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 trx_rw_min_trx_id () at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/trx0sys.ic:213
#11 lock_sec_rec_read_check_and_lock (duration=duration@entry=lock_duration_t::REGULAR, block=block@entry=0x7fddaa7ec730, rec=rec@entry=0x7fddab143cfb "0", index=index@entry=0x7fdb540455e8, offsets=0x7fdbb009ab10, sel_mode=sel_mode@entry=SELECT_ORDINARY, mode=LOCK_S, gap_mode=0, thr=0x7fdaf44f0420) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/lock/lock0lock.cc:5535
#12 0x00000000023447f5 in sel_set_rec_lock (pcur=pcur@entry=0x7fdaf44ef0e0, rec=rec@entry=0x7fddab143cfb "0", index=index@entry=0x7fdb540455e8, offsets=offsets@entry=0x7fdbb009ab10, sel_mode=SELECT_ORDINARY, mode=<optimized out>, type=0, thr=0x7fdaf44f0420, mtr=0x7fdbb009b150) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/row/row0sel.cc:1170
#13 0x0000000002349606 in row_search_mvcc (buf=buf@entry=0x7fdaf403aab8 "", mode=PAGE_CUR_GE, mode@entry=PAGE_CUR_UNSUPP, prebuilt=0x7fdaf44eeeb8, match_mode=match_mode@entry=1, direction=direction@entry=1) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/row/row0sel.cc:5225
#14 0x00000000021e9830 in ha_innobase::general_fetch (this=0x7fdaf40393f8, buf=0x7fdaf403aab8 "", direction=1, match_mode=1) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/ha_innodb.cc:10224
#15 0x0000000001131b5c in handler::ha_index_next_same (this=0x7fdaf40393f8, buf=0x7fdaf403aab8 "", key=0x7fdaf44f2870 "", keylen=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/handler.cc:3458
#16 0x0000000000eb6c0b in RefIterator<false>::Read (this=0x7fdaf44f2c00) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_executor.cc:3795
#17 0x0000000000fb4a6b in Query_expression::ExecuteIteratorQuery (this=0x7fdaf43e3280, thd=thd@entry=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1231
#18 0x0000000000fb4bac in Query_expression::execute (this=<optimized out>, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1284
#19 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fdaf43e6068, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#20 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fdaf405c460, first_level=first_level@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#21 0x0000000000e4b631 in sp_instr_stmt::exec_core (this=0x7fdaf43e61e8, thd=0x7fdaf405c460, nextp=0x7fdbb009cafc) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:966
#22 0x0000000000e4d699 in sp_lex_instr::reset_lex_and_exec_core (this=this@entry=0x7fdaf43e61e8, thd=thd@entry=0x7fdaf405c460, nextp=nextp@entry=0x7fdbb009cafc, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:441
#23 0x0000000000e4e6a4 in sp_lex_instr::validate_lex_and_execute_core (this=this@entry=0x7fdaf43e61e8, thd=thd@entry=0x7fdaf405c460, nextp=nextp@entry=0x7fdbb009cafc, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:727
#24 0x0000000000e4ff4c in sp_instr_stmt::execute (this=0x7fdaf43e61e8, thd=0x7fdaf405c460, nextp=0x7fdbb009cafc) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:892
#25 0x0000000000e44166 in sp_head::execute (this=this@entry=0x7fdaf40a0548, thd=thd@entry=0x7fdaf405c460, merge_da_on_success=merge_da_on_success@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2222
#26 0x0000000000e473f3 in sp_head::execute_procedure (this=this@entry=0x7fdaf40a0548, thd=thd@entry=0x7fdaf405c460, args=0x7fdaf4269da0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2871
#27 0x00000000013521e6 in Sql_cmd_call::execute_inner (this=0x7fdaf426a4e8, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_call.cc:235
#28 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fdaf426a4e8, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#29 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fdaf405c460, first_level=first_level@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#30 0x0000000000e4b631 in sp_instr_stmt::exec_core (this=0x7fdaf426a560, thd=0x7fdaf405c460, nextp=0x7fdbb009e59c) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:966
#31 0x0000000000e4d699 in sp_lex_instr::reset_lex_and_exec_core (this=this@entry=0x7fdaf426a560, thd=thd@entry=0x7fdaf405c460, nextp=nextp@entry=0x7fdbb009e59c, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:441
#32 0x0000000000e4e6a4 in sp_lex_instr::validate_lex_and_execute_core (this=this@entry=0x7fdaf426a560, thd=thd@entry=0x7fdaf405c460, nextp=nextp@entry=0x7fdbb009e59c, open_tables=open_tables@entry=false) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:727
#33 0x0000000000e4ff4c in sp_instr_stmt::execute (this=0x7fdaf426a560, thd=0x7fdaf405c460, nextp=0x7fdbb009e59c) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_instr.cc:892
#34 0x0000000000e44166 in sp_head::execute (this=this@entry=0x7fdaf41592e8, thd=thd@entry=0x7fdaf405c460, merge_da_on_success=merge_da_on_success@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2222
#35 0x0000000000e473f3 in sp_head::execute_procedure (this=this@entry=0x7fdaf41592e8, thd=thd@entry=0x7fdaf405c460, args=0x7fdaf4469ca8) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sp_head.cc:2871
#36 0x00000000013521e6 in Sql_cmd_call::execute_inner (this=0x7fdaf446a150, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_call.cc:235
#37 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fdaf446a150, thd=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#38 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fdaf405c460, first_level=first_level@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#39 0x0000000000ef6045 in dispatch_sql_command (thd=thd@entry=0x7fdaf405c460, parser_state=parser_state@entry=0x7fdbb00a0640) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:5033
#40 0x0000000000ef79b5 in dispatch_command (thd=thd@entry=0x7fdaf405c460, com_data=com_data@entry=0x7fdbb00a0d30, command=COM_QUERY) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1863
#41 0x0000000000ef921c in do_command (thd=thd@entry=0x7fdaf405c460) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1342
#42 0x000000000101d6f0 in handle_connection (arg=arg@entry=0x6df5380) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/conn_handler/connection_handler_per_thread.cc:301
#43 0x000000000262b551 in pfs_spawn_thread (arg=0x73076d0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs.cc:2898
#44 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#45 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
Thread 74 (LWP 2894810):
#0  0x00007fde1043fa1c in pthread_cond_wait () from /usr/lib64/libpthread.so.0
#1  0x00000000022bbc10 in os_event::wait (this=0x7fddfc329910) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:187
#2  os_event::wait_low (this=0x7fddfc329910, reset_sig_count=2) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:370
#3  0x00000000022bbffa in os_event_wait_low (event=<optimized out>, reset_sig_count=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/os/os0event.cc:602
#4  0x0000000002380ab7 in sync_array_wait_event (arr=arr@entry=0x7fddfc00b5d0, cell=@0x7fdb4071da70: 0x7fde028460f0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/sync/sync0arr.cc:326
#5  0x00000000021de298 in TTASEventMutex<GenericPolicy>::wait (this=this@entry=0x7fddfc32c958, filename=filename@entry=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", line=line@entry=856, spin=spin@entry=4) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.ic:210
#6  0x00000000021de409 in TTASEventMutex<GenericPolicy>::spin_and_try_lock (this=0x7fddfc32c958, max_spins=60, max_delay=6, filename=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", line=856) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:520
#7  0x00000000022303d1 in TTASEventMutex<GenericPolicy>::enter (line=856, filename=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", max_delay=6, max_spins=30, this=0x7fddfc32c958) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:414
#8  PolicyMutex<TTASEventMutex<GenericPolicy> >::enter (line=856, name=0x3092c48 "/app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc", n_delay=6, n_spins=30, this=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ib0mutex.h:615
#9  mutex_enter_inline<PolicyMutex<TTASEventMutex<GenericPolicy> > > (loc=..., m=<optimized out>) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/include/ut0mutex.h:114
#10 Innodb_data_lock_wait_iterator::scan (this=0x7fda60554750, container=0x7fdad5014fb0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/innobase/handler/p_s.cc:856
#11 0x000000000265f72e in table_data_lock_waits::rnd_next (this=0x7fdad5014e60) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/table_data_lock_waits.cc:185
#12 0x0000000002624cae in ha_perfschema::rnd_next (this=0x7fdb5c0fe5a8, buf=0x7fdb5c0ffc38 "\377") at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/ha_perfschema.cc:1678
#13 0x00000000011306fc in handler::ha_rnd_next (this=0x7fdb5c0fe5a8, buf=0x7fdb5c0ffc38 "\377") at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/handler.cc:2985
#14 0x0000000000e305dd in TableScanIterator::Read (this=0x7fdad500fd70) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/records.cc:361
#15 0x0000000000fb4a6b in Query_expression::ExecuteIteratorQuery (this=0x7fdad500c208, thd=thd@entry=0x7fdad55ac090) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1231
#16 0x0000000000fb4bac in Query_expression::execute (this=<optimized out>, thd=0x7fdad55ac090) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_union.cc:1284
#17 0x0000000000f51144 in Sql_cmd_dml::execute (this=0x7fdad500e248, thd=0x7fdad55ac090) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_select.cc:574
#18 0x0000000000ef27e2 in mysql_execute_command (thd=thd@entry=0x7fdad55ac090, first_level=first_level@entry=true) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:4436
#19 0x0000000000ef6045 in dispatch_sql_command (thd=thd@entry=0x7fdad55ac090, parser_state=parser_state@entry=0x7fdb4071f640) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:5033
#20 0x0000000000ef79b5 in dispatch_command (thd=thd@entry=0x7fdad55ac090, com_data=com_data@entry=0x7fdb4071fd30, command=COM_QUERY) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1863
#21 0x0000000000ef921c in do_command (thd=thd@entry=0x7fdad55ac090) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/sql_parse.cc:1342
#22 0x000000000101d6f0 in handle_connection (arg=arg@entry=0x6bdc4d0) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/sql/conn_handler/connection_handler_per_thread.cc:301
#23 0x000000000262b551 in pfs_spawn_thread (arg=0x6b48110) at /app/mysql_8.0.26_x86/mysql-8.0.26_test/mysql-8.0.26/storage/perfschema/pfs.cc:2898
#24 0x00007fde10439f2b in ?? () from /usr/lib64/libpthread.so.0
#25 0x00007fde0fc106bf in clone () from /usr/lib64/libc.so.6
``` 

只有Thread 21不在 error log的锁列表中. 

情况也合理, 因为Thread 21就是打印 error log信息的函数. 卡主以后就不再打印innodb monitor信息了, 也就看不到锁列表了

# 结论

通过复现, 在mysql debug版本上能报出deadlock错误, 定位到问题: 

[附件: Bug 35068461 (2024_8_20 11_49_23).html] [附件: Document 2986699.1 (2024_8_20 11_49_37).html] 

问题简述: 在performance_schema.data_lock的表的错误处理中, C++的异常被try...catch...后, 转换成了一个MySQL的报错, 但没有对C++的异常后的现场 (未解开的锁 等) 进行恢复处理. 就会出现各种奇异的错误. 

# 可参考的问题诊断手法

  - 规范MySQL卡住时应当获取的信息列表
    - 多次获取堆栈
    - 打印火焰图
    - 一定要获取error log
    - 可以获取strace
  - 将堆栈按线程进行分割, 使用grep来进行线程的过滤筛选
  - 将error log中的锁列表 与 堆栈中的锁 进行对照, 判断是否存在活锁
