---
title: 20210929 - 测试 jemalloc
confluence_page_id: 1343772
created_at: 2021-09-29T09:34:03+00:00
updated_at: 2021-10-02T06:19:24+00:00
---

# 基础

jemalloc需自己编译, configure是增加 --enable-prof 参数, 启动perf功能

jemalloc MySQL启动命令: 

```
LD_PRELOAD=/tmp/libjemalloc.so.2 MALLOC_CONF="prof:true,lg_prof_interval:26" /data/liukaiyang/mysql/8.0.19/bin/mysqld --defaults-file=/data/liukaiyang/sandboxes/msb_8_0_19/my.sandbox.cnf --basedir=/data/liukaiyang/sandboxes/msb_8_0_19 --datadir=/data/liukaiyang/sandboxes/msb_8_0_19/data --plugin-dir=/usr/local/mysql/lib/plugin --user=root --log-error=/data/liukaiyang/sandboxes/msb_8_0_19/data/msandbox.err --open-files-limit=65535 --pid-file=/data/liukaiyang/sandboxes/msb_8_0_19/data/mysql_sandbox8019.pid --socket=/tmp/mysql_sandbox8019.sock --port=8019
``` 

lg_prof_interval: 每2^26字节分配, 进行一次dump

还有其他参数可控制dump行为, 但不提供按信号dump

打印svg命令: 

```
/tmp/jeprof --svg /data/liukaiyang/mysql/8.0.19/bin/mysqld jeprof.65881.57.i57.heap > /tmp/1.svg
``` 

导出的svg中:

  1. 百分比是错误的
  2. 并没有记录mmap分配的内存量

# 是否确实不记录mmap分配的内存量

探索tcmalloc和jemalloc在此行为上的差异

在开启MySQL时, 立刻使用perf观测MySQL: 

```
perf record -e syscalls:sys_enter_mmap -p $(pgrep mysqld) --call-graph dwarf
``` 

当使用tcmalloc时, MySQL直接使用mmap分出去的内存堆栈为: 

```
mysqld 75949 [000] 1138495.128453: syscalls:sys_enter_mmap: addr: 0x00000000, len: 0x08300000, prot
            7f4b0bc6ad19 syscall+0x19 (/usr/lib64/libc-2.17.so)
            7f4b0ddf6a35 do_mmap64+0x4d (/opt/gperftools/libtcmalloc.so)
            7f4b0de145bb mmap64+0x95 (/opt/gperftools/libtcmalloc.so)
                 18fcf4d os_mem_alloc_large+0x1d8 (/data/liukaiyang/mysql/8.0.19/bin/mysqld)
                 19b2b5c buf_pool_t::allocate_chunk+0x36 (/data/liukaiyang/mysql/8.0.19/bin/mysqld)
                 19b4557 buf_chunk_init+0x57 (/data/liukaiyang/mysql/8.0.19/bin/mysqld)
                 19b4b93 buf_pool_create+0x3b6 (/data/liukaiyang/mysql/8.0.19/bin/mysqld)
                 1c1acae execute_native_thread_routine+0xe (/data/liukaiyang/mysql/8.0.19/bin/mysql
            7f4b0db62ea4 start_thread+0xc4 (/usr/lib64/libpthread-2.17.so)
            7f4b0bc709fc __clone+0x6c (/usr/lib64/libc-2.17.so)
``` 

mmap会由tcmalloc转发

当使用jemalloc时, 内存堆栈为: 

```
mysqld 75700 [000] 1138300.220861: syscalls:sys_enter_mmap: addr: 0x00000000, len: 0x08300000, prot
            7fb58b9f0eba __mmap64+0x3a (/usr/lib64/libc-2.17.so)
                 18fcf4d os_mem_alloc_large+0x1d8 (/data/liukaiyang/mysql/8.0.19/bin/mysqld)
                 19b2b5c buf_pool_t::allocate_chunk+0x36 (/data/liukaiyang/mysql/8.0.19/bin/mysqld)
                 19b4557 buf_chunk_init+0x57 (/data/liukaiyang/mysql/8.0.19/bin/mysqld)
                 19b4b93 buf_pool_create+0x3b6 (/data/liukaiyang/mysql/8.0.19/bin/mysqld)
                 1c1acae execute_native_thread_routine+0xe (/data/liukaiyang/mysql/8.0.19/bin/mysql
            7fb58d8e8ea4 start_thread+0xc4 (/usr/lib64/libpthread-2.17.so)
            7fb58b9f69fc __clone+0x6c (/usr/lib64/libc-2.17.so)
``` 

mmap不经过jemalloc, jemalloc无法统计mmap分配的内存量

# 性能

jemalloc开启统计时, 性能与glibc几乎一致, 未见明显衰减
