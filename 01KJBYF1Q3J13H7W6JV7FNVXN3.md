---
title: 20211008 - 如何调试jemalloc对unwind库的调用
confluence_page_id: 1343780
created_at: 2021-10-08T08:00:25+00:00
updated_at: 2021-10-08T10:02:10+00:00
---

用jemalloc启动MySQL

```
LD_PRELOAD=/tmp/libjemalloc.so.2 MALLOC_CONF="prof:true,lg_prof_interval:26" /data/liukaiyang/mysql/8.0.19/bin/mysqld --defaults-file=/data/liukaiyang/sandboxes/msb_8_0_19/my.sandbox.cnf --basedir=/data/liukaiyang/sandboxes/msb_8_0_19 --datadir=/data/liukaiyang/sandboxes/msb_8_0_19/data --plugin-dir=/usr/local/mysql/lib/plugin --user=root --log-error=/data/liukaiyang/sandboxes/msb_8_0_19/data/msandbox.err --open-files-limit=65535 --pid-file=/data/liukaiyang/sandboxes/msb_8_0_19/data/mysql_sandbox8019.pid --socket=/tmp/mysql_sandbox8019.sock --port=8019
``` 

我们需要调试 jemalloc 对 unwind 库的调用

尝试: ltrace, ltrace不支持在共享库间 监听调用, 不可用

尝试: latrace, 报错SIGSEGV, 不可用

# gdb 调试jemalloc的方法: 

输入以下gdb指令: 

```
set exec-wrapper env LD_PRELOAD=/tmp/libjemalloc.so.2 MALLOC_CONF="prof:true,lg_prof_interval:26"

file /data/liukaiyang/mysql/8.0.19/bin/mysqld

set args --defaults-file=/data/liukaiyang/sandboxes/msb_8_0_19/my.sandbox.cnf --basedir=/data/liukaiyang/sandboxes/msb_8_0_19 --datadir=/data/liukaiyang/sandboxes/msb_8_0_19/data --plugin-dir=/usr/local/mysql/lib/plugin --user=root --log-error=/data/liukaiyang/sandboxes/msb_8_0_19/data/msandbox.err --open-files-limit=65535 --pid-file=/data/liukaiyang/sandboxes/msb_8_0_19/data/mysql_sandbox8019.pid --socket=/tmp/mysql_sandbox8019.sock --port=8019
 
b _Unwind_Backtrace
r
``` 

可监控到相关调用堆栈, 举例: 

```
#0  0x00007ffff5af8f94 in _Unwind_Backtrace ()
   from /lib64/libgcc_s.so.1
#1  0x00007ffff797aee2 in je_prof_backtrace (
    bt=bt@entry=0x7fffffffd8f0) at src/prof.c:635
#2  0x00007ffff7930e83 in prof_alloc_prep (update=true,
    je_prof_active=<optimized out>, usize=28672, tsd=0x7ffff7fe2750)
    at include/jemalloc/internal/prof_inlines_b.h:158
#3  imalloc_body (tsd=0x7ffff7fe2750, dopts=<synthetic pointer>,
    sopts=<synthetic pointer>) at src/jemalloc.c:2116
#4  imalloc (dopts=<synthetic pointer>, sopts=<synthetic pointer>)
    at src/jemalloc.c:2258
#5  posix_memalign (memptr=0x7fffffffd958, alignment=64,
    size=<optimized out>) at src/jemalloc.c:2416
#6  0x0000000001ac4b57 in pfs_malloc(PFS_builtin_memory_class*, unsigned long, int) ()
#7  0x0000000001ac4c1b in pfs_malloc_array(PFS_builtin_memory_class*, unsigned long, unsigned long, int) ()
#8  0x0000000001acd797 in init_file_class(unsigned int) ()
#9  0x0000000001ad1942 in initialize_performance_schema(PFS_global_param*, PSI_thread_bootstrap**, PSI_mutex_bootstrap**, PSI_rwlock_bootstrap**, PSI_cond_bootstrap**, PSI_file_bootstrap**, PSI_socket_bootstrap**, PSI_table_bootstrap**, PSI_mdl_bootstrap**, PSI_idle_bootstrap**, PSI_stage_bootstrap**, PSI_statement_bootstrap**, PSI_transaction_bootstrap**, PSI_memory_bootstrap**, PSI_error_bootstrap**, PSI_data_lock_bootstrap**, PSI_system_bootstrap**) ()
#10 0x0000000000ee6252 in mysqld_main(int, char**) ()
#11 0x00007ffff573d555 in __libc_start_main () from /lib64/libc.so.6
#12 0x0000000000ed9237 in _start ()
``` 

# gdb 调试tcmalloc的方法: 

注意: 通过TCMALLOC_STACKTRACE_METHOD 指定 libgcc 为追踪方法

```
set exec-wrapper env LD_PRELOAD=/usr/lib64/libtcmalloc.so.4 HEAPPROFILE=/data/liukaiyang/mysql-profile-insert HEAP_PROFILE_INUSE_INTERVAL=0 HEAP_PROFILE_ALLOCATION_INTERVAL=0 HEAPPROFILESIGNAL=21 HEAP_PROFILE_MMAP=true TCMALLOC_STACKTRACE_METHOD=libgcc TCMALLOC_STACKTRACE_METHOD_VERBOSE=true

file /data/liukaiyang/mysql/8.0.19/bin/mysqld

set args --defaults-file=/data/liukaiyang/sandboxes/msb_8_0_19/my.sandbox.cnf --basedir=/data/liukaiyang/sandboxes/msb_8_0_19 --datadir=/data/liukaiyang/sandboxes/msb_8_0_19/data --plugin-dir=/usr/local/mysql/lib/plugin --user=root --log-error=/data/liukaiyang/sandboxes/msb_8_0_19/data/msandbox.err --open-files-limit=65535 --pid-file=/data/liukaiyang/sandboxes/msb_8_0_19/data/mysql_sandbox8019.pid --socket=/tmp/mysql_sandbox8019.sock --port=8019
 
b _Unwind_Backtrace
 
r
``` 

可获取堆栈: 

```
#0  0x00007ffff5bb2f94 in _Unwind_Backtrace () from /lib64/libgcc_s.so.1
#1  0x00007ffff7a0edba in GetStackTrace_libgcc(void**, int, int) ()
   from /usr/lib64/libtcmalloc.so.4
#2  0x00007ffff7a0f394 in GetStackTrace(void**, int, int) ()
   from /usr/lib64/libtcmalloc.so.4
#3  0x00007ffff7a045c6 in MallocHook_GetCallerStackTrace ()
   from /usr/lib64/libtcmalloc.so.4
#4  0x00007ffff7a095a5 in NewHook () from /usr/lib64/libtcmalloc.so.4
#5  0x00007ffff7a049cc in MallocHook::InvokeNewHookSlow(void const*, unsigned long)
    () from /usr/lib64/libtcmalloc.so.4
#6  0x00007ffff7a10cd3 in tcmalloc::allocate_full_malloc_oom(unsigned long) ()
   from /usr/lib64/libtcmalloc.so.4
#7  0x00000000018007f3 in my_malloc(unsigned int, unsigned long, int) ()
#8  0x00000000016cd8b7 in std::__detail::_Hashtable_alloc<Malloc_allocator<std::__detail::_Hash_node<std::pair<binary_log::Uuid const, std::unique_ptr<Sid_map::Node, My_free_deleter> >, true> > >::_M_allocate_buckets(unsigned long) ()
#9  0x00000000016cd99c in Sid_map::Sid_map(Checkable_rwlock*) ()
#10 0x00000000019d6261 in Clone_persist_gtid::flush_gtids(THD*) ()
#11 0x00000000019d6634 in Clone_persist_gtid::periodic_write() ()
#12 0x00000000019d5235 in std::thread::_State_impl<std::thread::_Invoker<std::tuple<Runnable, void (*)(Clone_persist_gtid*), Clone_persist_gtid*> > >::_M_run() ()
#13 0x0000000001c1acaf in execute_native_thread_routine ()
#14 0x00007ffff77c5ea5 in start_thread () from /lib64/libpthread.so.0
#15 0x00007ffff58d39fd in clone () from /lib64/libc.so.6
```
