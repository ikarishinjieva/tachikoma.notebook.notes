---
title: 20220302 - 工行MySQL crash
confluence_page_id: 1736903
created_at: 2022-03-02T09:51:39+00:00
updated_at: 2022-03-03T16:46:21+00:00
---

工单: BEIJ-2241

崩溃堆栈: 

![image2022-3-2 17:32:29.png](/assets/01KJBYHCENW7J7KPJMGXJGAK4S/image2022-3-2%2017%3A32%3A29.png)

![image2022-3-2 17:4:11.png](/assets/01KJBYHCENW7J7KPJMGXJGAK4S/image2022-3-2%2017%3A4%3A11.png)

注意点: 这个MySQL实例是工行独自编译, 运行在信创os上

堆栈解析: 

```
/data/app/mysql/bin/mysqld(_Z19page_cur_delete_recP10page_cur_tPK12dict_index_tPKmP5mtr_t+0x1fc)[0xf14e6c]
mach0data.ic:88

/data/app/mysql/bin/mysqld(_Z30btr_cur_optimistic_delete_funcP9btr_cur_tP5mtr_t+0x208)[0x1014e58]
Breakpoint 1 at 0x1014e58: file /data1/xk/install/mysql_5.7.27_fix1/storage/innobase/btr/btr0cur.cc:5158.

/data/app/mysql/bin/mysqld[0xf76684]
Breakpoint 1 at 0xf76684: file /data1/xk/install/mysql_5.7.27_fix1/storage/innobase/row/row0purge.cc:168.

/data/app/mysql/bin/mysqld(_Z14row_purge_stepP9que_thr_t+0x47c)[0xf78294]
Breakpoint 1 at 0xf78294: file /data1/xk/install/mysql_5.7.27_fix1/storage/innobase/row/row0purge.cc:215.

/data/app/mysql/bin/mysqld(_Z15que_run_threadsP9que_thr_t+0xa6c)[0xf345b4]
Breakpoint 1 at 0xf345b4: file /data1/xk/install/mysql_5.7.27_fix1/storage/innobase/que/que0que.cc:1049.
``` 

断点第一层堆栈和第二层堆栈连接不上, 怀疑是编译参数开启了高级别优化, 将堆栈信息挤压掉了

复现断点: 

在一张表中插入数据后删除, 断点打在如下位置: (_Z19page_cur_delete_recP10page_cur_tPK12dict_index_tPKmP5mtr_t+0x1fc)

可复现断点堆栈: 

```
(gdb) bt
#0  0x0000000001074ebc in mach_read_from_2 (b=0x7f3b038ec061 "")
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/include/mach0data.ic:88
#1  rec_get_next_offs (rec=0x7f3b038ec063 "infimum", comp=32768)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/include/rem0rec.ic:326
#2  page_rec_get_next_low (comp=32768, rec=0x7f3b038ec063 "infimum")
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/include/page0page.ic:853
#3  page_rec_get_next (rec=0x7f3b038ec063 "infimum")
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/include/page0page.ic:882
#4  page_cur_delete_rec (cursor=0x2de8310, index=0x7f3ac8039cf8,
    offsets=0x7f3af37fd0b0, mtr=<optimized out>)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/page/page0cur.cc:2661
#5  0x000000000117337d in btr_cur_optimistic_delete_func (cursor=0x2de8308,
    mtr=0x7f3af37fd440)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/btr/btr0cur.cc:5153
#6  0x00000000010d9388 in row_purge_remove_clust_if_poss_low (
    node=0x2de8270, mode=2)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/row/row0purge.cc:169
#7  0x00000000010dba51 in row_purge_remove_clust_if_poss (node=0x2de8270)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/row/row0purge.cc:215
#8  row_purge_del_mark (node=0x2de8270)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/row/row0purge.cc:667
#9  row_purge_record_func (updated_extern=<optimized out>,
    undo_rec=0x2de8718 "\001T\016\001(", node=0x2de8270)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/row/row0purge.cc:990
#10 row_purge (thr=0x2de81a8, undo_rec=0x2de8718 "\001T\016\001(",
    node=0x2de8270)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/row/row0purge.cc:1046
#11 row_purge_step (thr=0x2de81a8)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/row/row0purge.cc:1125
#12 0x0000000001091d7f in que_thr_step (thr=0x2de81a8)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/que/que0que.cc:1049
#13 que_run_threads_low (thr=0x2de81a8)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/que/que0que.cc:1111
#14 que_run_threads (thr=<optimized out>)
    at /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/que/que0que.cc:1151
``` 

崩溃信号为 signal 11, 应当是 byte数组地址错误

![image2022-3-2 18:11:40.png](/assets/01KJBYHCENW7J7KPJMGXJGAK4S/image2022-3-2%2018%3A11%3A40.png)

byte数组地址为 rec -2

![image2022-3-2 18:12:16.png](/assets/01KJBYHCENW7J7KPJMGXJGAK4S/image2022-3-2%2018%3A12%3A16.png)

rec来自于 page_cur_delete_rec ([page0cur.cc](<http://page0cur.cc>):2661) 的遍历

![image2022-3-2 18:12:58.png](/assets/01KJBYHCENW7J7KPJMGXJGAK4S/image2022-3-2%2018%3A12%3A58.png)

关于 purge 的逻辑参考: 

  - <https://developer.aliyun.com/article/646471>
  - <http://mysql.taobao.org/monthly/2018/03/01/>
