---
title: 20211009 - 监控MySQL内存的命令
confluence_page_id: 1343799
created_at: 2021-10-09T03:00:07+00:00
updated_at: 2021-10-09T03:00:26+00:00
---

# 只监控mmap

使用tcmalloc, 只监控mmap

```
LD_PRELOAD="/usr/lib64/libtcmalloc.so.4" HEAPPROFILE=/data/liukaiyang/mysql-profile-insert HEAP_PROFILE_INUSE_INTERVAL=0 HEAP_PROFILE_ALLOCATION_INTERVAL=0 HEAPPROFILESIGNAL=21 HEAP_PROFILE_ONLY_MMAP=true /data/liukaiyang/mysql/8.0.19/bin/mysqld --defaults-file=/data/liukaiyang/sandboxes/msb_8_0_19/my.sandbox.cnf --basedir=/data/liukaiyang/sandboxes/msb_8_0_19 --datadir=/data/liukaiyang/sandboxes/msb_8_0_19/data --plugin-dir=/usr/local/mysql/lib/plugin --user=root --log-error=/data/liukaiyang/sandboxes/msb_8_0_19/data/msandbox.err --open-files-limit=65535 --pid-file=/data/liukaiyang/sandboxes/msb_8_0_19/data/mysql_sandbox8019.pid --socket=/tmp/mysql_sandbox8019.sock --port=8019
``` 

监控力度粗, 会有少量偏差

# 同时监控mmap和malloc

同时使用jemalloc和tcmalloc

```
LD_PRELOAD="/tmp/libjemalloc.so.2 /usr/lib64/libtcmalloc.so.4 " HEAPPROFILE=/data/liukaiyang/mysql-profile-insert HEAP_PROFILE_INUSE_INTERVAL=0 HEAP_PROFILE_ALLOCATION_INTERVAL=0 HEAPPROFILESIGNAL=21 HEAP_PROFILE_ONLY_MMAP=true MALLOC_CONF="prof:true,lg_prof_interval:26" /data/liukaiyang/mysql/8.0.19/bin/mysqld --defaults-file=/data/liukaiyang/sandboxes/msb_8_0_19/my.sandbox.cnf --basedir=/data/liukaiyang/sandboxes/msb_8_0_19 --datadir=/data/liukaiyang/sandboxes/msb_8_0_19/data --plugin-dir=/usr/local/mysql/lib/plugin --user=root --log-error=/data/liukaiyang/sandboxes/msb_8_0_19/data/msandbox.err --open-files-limit=65535 --pid-file=/data/liukaiyang/sandboxes/msb_8_0_19/data/mysql_sandbox8019.pid --socket=/tmp/mysql_sandbox8019.sock --port=8019
``` 

可分别得到mmap和malloc的结果, 但mmap结果中, 会包含部分malloc从jemalloc分走内存的结果 . 

另外, 混合用法pprof会报错: 

```
inuse count was 0 but inuse bytes was xxxxxx
``` 

可以将heap.prof文件中的xxxxxx行找出, 将inuse为0的行删除.
