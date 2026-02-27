---
title: 20210819 - MySQL 内存, 除了p_s统计的, 还有哪些 3 - tcmalloc
confluence_page_id: 1343650
created_at: 2021-08-20T06:42:59+00:00
updated_at: 2021-08-27T06:24:26+00:00
---

# 背景

使用 tcmalloc 进行观察: <http://10.186.18.11/confluence/pages/viewpage.action?pageId=31104463>

MySQL 版本: 8.0.25

获取 svg: [profile001.svg](/assets/01KJBYED7JBE2YBE7Y4YY213H6/profile001.svg)

# 技巧

增加 HEAPPROFILESIGNAL=21, 通过SIGTTIN可以触发HEAP

# 分析一些分配条目

  1. log_allocate_buffer函数
     1. 与 innodb_log_buffer_size 参数有关
  2. Link_buf 构造
     1. 参考: <http://mysql.taobao.org/monthly/2020/06/04/>
     2. 一共两个Link_buf
        1. log.recent_written: 
           1. 用途是: 允许事务并发写入redo log. log writer 需要获取连续的redo log段以写入. Link_buf用于查找redo log连续到了哪个位置.
           2. 代码: 

```
/** Number of slots in a small buffer, which is used to allow concurrent
writes to log buffer. The slots are addressed by LSN values modulo number
of the slots. */
extern ulong srv_log_recent_written_size;
``` 
           3. 默认值: 1024 * 1024 字节, MySQL层不可配置
        2. log.recent_closed: 
           1. 代码: 

```
/** Number of slots in a small buffer, which is used to break requirement
for total order of dirty pages, when they are added to flush lists.
The slots are addressed by LSN values modulo number of the slots. */
extern ulong srv_log_recent_closed_size;
``` 
           2. 用途是: 允许脏页写入flush list时产生并发, 而刷脏时找到连续的段进行刷脏
           3. 默认值: 2 * 1024 * 1024 字节, MySQL层不可配置
     3. 总大小: 在内存分配图中为 24576 kB, 理论值为 (1M+2M) * 8 = 24M, (8 是Distance的大小), 见如下代码: 

![image2021-8-20 18:8:2.png](/assets/01KJBYED7JBE2YBE7Y4YY213H6/image2021-8-20%2018%3A8%3A2.png)

  3. Pages::add
     1. 为double write buffer的机制, recv::Pages::add
     2. 从double write buffer file中读取文件内容, 每次申请Buffer大小为一页, Buffer默认多配置一页, 所以内存消耗为double write buffer file的大小的两倍
     3. 参考: [20210821 - MySQL double write buffer 相关]
  4. os_create_block_cache
     1. 申请文件系统用的Block缓存
        1. 每个Block大小为 UNIV_PAGE_SIZE * 1.3 = 20.8 kB
        2. 一共128个Block
        3. 如果cache不足, 可临时申请
     2. Block缓存目前用于引擎的加解密
  5. ...

# 对比os的统计和tcmalloc的统计差异

os的统计: 

```
root@ubuntu:/tmp# ps aux | grep mysqld
root     21812  6.0  2.5 889480 426716 pts/1   Sl   16:15   0:15 /root/opt/mysql/8.0.25/bin/mysqld --defaults-file=/root/sandboxes/test-memory/my.sandbox.cnf --basedir=/root/opt/mysql/8.0.25 --datadir=/root/sandboxes/test-memory/data --plugin-dir=/root/opt/mysql/8.0.25/lib/plugin --user=root --log-error=/root/sandboxes/test-memory/data/msandbox.err --pid-file=/root/sandboxes/test-memory/data/mysql_sandbox8025.pid --socket=/tmp/mysql_sandbox8025.sock --port=8025
``` 

RSS = 426716kB

tcmalloc的统计值为 330938kB: 

![image2021-8-22 0:19:45.png](/assets/01KJBYED7JBE2YBE7Y4YY213H6/image2021-8-22%200%3A19%3A45.png)

查看smaps: [smaps.txt](/assets/01KJBYED7JBE2YBE7Y4YY213H6/smaps.txt)

其中heap的大小为: Rss为 344468kB

```
05c9d000-1c0a8000 rw-p 00000000 00:00 0                                  [heap]
Size:             364588 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
Rss:              344468 kB
Pss:              344468 kB
Shared_Clean:          0 kB
Shared_Dirty:          0 kB
Private_Clean:     20564 kB
Private_Dirty:    323904 kB
Referenced:       322856 kB
Anonymous:        344468 kB
LazyFree:          20564 kB
AnonHugePages:         0 kB
ShmemPmdMapped:        0 kB
Shared_Hugetlb:        0 kB
Private_Hugetlb:       0 kB
Swap:                  0 kB
SwapPss:               0 kB
Locked:                0 kB
VmFlags: rd wr mr mw me ac sd
``` 

smaps其中Rss总数为: 427696 kB

smaps中的代码段特征为带有执行权限 (x)

```
00400000-03874000 r-xp 00000000 fc:01 1039240                            /root/opt/mysql/8.0.25/bin/mysqld
Size:              53712 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
Rss:               25816 kB
Pss:               25814 kB
Shared_Clean:          4 kB
Shared_Dirty:          0 kB
Private_Clean:     25812 kB
Private_Dirty:         0 kB
Referenced:        25816 kB
Anonymous:             0 kB
LazyFree:              0 kB
AnonHugePages:         0 kB
ShmemPmdMapped:        0 kB
Shared_Hugetlb:        0 kB
Private_Hugetlb:       0 kB
Swap:                  0 kB
SwapPss:               0 kB
Locked:                0 kB
VmFlags: rd ex mr mw me dw sd
``` 

# 验证其他的smaps

从工单 <https://support.actionsky.com/service_desk/browse/BEIJ-1632> 找到smaps输出: [smaps.txt](/assets/01KJBYED7JBE2YBE7Y4YY213H6/smaps.txt)

其中Rss 为 42.49 GB: 

```
┌────[tachikoma@miao]───────[19:40:49]───────[/tmp]────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
└──> cat ~/Downloads/smaps.txt | egrep '^Rss' | awk '{s+=$2} END {print s}'
44556088
``` 

## 但堆内存只有 1G+

```
┌────[tachikoma@miao]───────[19:40:51]───────[/tmp]────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
└──> cat ~/Downloads/smaps.txt | grep -A 15 'heap'
031e5000-4817b000 rw-p 00000000 00:00 0                                  [heap]
Size:            1130072 kB
Rss:             1130072 kB
Pss:             1130072 kB
Shared_Clean:          0 kB
Shared_Dirty:          0 kB
Private_Clean:         0 kB
Private_Dirty:   1130072 kB
Referenced:      1089064 kB
Anonymous:       1130072 kB
AnonHugePages:    686080 kB
Swap:                  0 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
Locked:                0 kB
VmFlags: rd wr mr mw me ac sd
``` 

参考: [20210825 - Linux smaps的相关文档]

malloc使用mmap时, 内存段不会标记[heap]
