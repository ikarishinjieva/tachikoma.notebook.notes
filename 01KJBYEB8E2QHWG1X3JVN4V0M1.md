---
title: 20210819 - Memory Leak (and Growth) Flame Graphs 文章解析
confluence_page_id: 1343635
created_at: 2021-08-19T04:19:19+00:00
updated_at: 2021-08-19T16:35:59+00:00
---

文章地址: <https://www.brendangregg.com/FlameGraphs/memoryflamegraphs.html>

# 其他诊断内存分配的手段

  1. memcheck
  2. libtcmalloc
     1. <http://goog-perftools.sourceforge.net/doc/heap_profiler.html>

# 追踪Allocator的malloc/free

Allocator的函数 操作的是 虚拟内存, 而不是RSS

对于kernel >= 4.9, 可使用bcc的stackcount

特点是: 离应用最近, 而追踪代价最高

# 追踪 brk 系统调用

brk 系统调用的作用: change data segment size

用于追踪虚拟内存的扩大

特点是: 追踪粒度不细, 仅追踪数据段的变更, 但追踪代价低

可使用 perf 或 bcc

# 追踪 mmap 系统调用

mmap 系统调用的作用: 分配大内存, 或 由Allocator调用分配大内存

用于追踪虚拟内存的扩大

特点是: 追踪粒度不细, 但追踪代价低

可使用 perf 或 bcc

# 追踪缺页异常

缺页异常 追踪物理内存的使用, 追踪代价低

可使用 perf 或 bcc

# libtcmalloc

参考: <http://goog-perftools.sourceforge.net/doc/heap_profiler.html>

```
apt install google-perftools
 
LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libtcmalloc_and_profiler.so.4 HEAPPROFILE=/tmp/profile /root/opt/mysql/8.0.25/bin/mysqld --defaults-file=/root/sandboxes/rsandbox_8_0_25/master/my.sandbox.cnf --basedir=/root/opt/mysql/8.0.25 --datadir=/root/sandboxes/rsandbox_8_0_25/master/data --plugin-dir=/root/opt/mysql/8.0.25/lib/plugin --user=root --log-error=/root/sandboxes/rsandbox_8_0_25/master/data/msandbox.err --pid-file=/root/sandboxes/rsandbox_8_0_25/master/data/mysql_sandbox21526.pid --socket=/tmp/mysql_sandbox21526.sock --port=21526
``` 

TODO: 查看 /tmp/profile.*
