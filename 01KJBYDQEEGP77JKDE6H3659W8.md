---
title: 20210718 - 组提交性能问题诊断
confluence_page_id: 1343505
created_at: 2021-07-18T02:09:45+00:00
updated_at: 2021-08-05T15:25:31+00:00
---

# 工单

<https://support.actionsky.com/service_desk/browse/SHAI-3425>

# 现象

开启binlog的trace log, 结果如下: 

```
2021-07-14T13:34:44.319508+08:00 1119047 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28578784) in entry(11121)
2021-07-14T13:34:44.890515+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28578784) in entry(11121)
2021-07-14T13:34:44.890519+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28578784) sync(1), repl(1)
2021-07-14T13:34:44.890991+08:00 1118887 [Note] ActiveTranx::::clear_active_tranx_nodes: cleared 1 nodes back until pos (mysql-bin.105618, 28578784)
2021-07-14T13:34:44.891074+08:00 0 [Note] ReplSemiSyncMaster::reportReplyPacket: Got reply(mysql-bin.105618, 28578784) from server 859389176
2021-07-14T13:34:44.891119+08:00 0 [Note] ReplSemiSyncMaster::reportReplyBinlog: Got reply at (mysql-bin.105618, 28578784)
``` 

  
延迟出现在 ActiveTranx:insert_tranx_node 和 ActiveTranx::is_tranx_end_pos 之间

探索代码中两行日志的代码位置: 

```
第一行日志
- binlog.after_flush
	- repl_semi_report_binlog_update
		- writeTranxInBinlog
			- insert_tranx_node

 
第二行日志
- Binlog_sender::send_events
	- repl_semi_before_send_event
		- updateSyncHeader
			- is_tranx_end_pos
	- send binlog
``` 

gdb探索两行日志的代码位置: 

```
第一行日志: 
 
#0  ActiveTranx::insert_tranx_node (this=0x4954540, log_file_name=0x2debf02 <mysql_bin_log+66> "mysql-bin.000017", log_file_pos=633)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/plugin/semisync/semisync_master.cc:138
#1  0x00007f407c38ecf8 in ReplSemiSyncMaster::writeTranxInBinlog (this=0x7f407c59b600 <repl_semisync>, log_file_name=0x2debf02 <mysql_bin_log+66> "mysql-bin.000017",
    log_file_pos=633) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/plugin/semisync/semisync_master.cc:1188
#2  0x00007f407c3907e5 in repl_semi_report_binlog_update (param=0x7f407c2f8b80, log_file=0x2debf02 <mysql_bin_log+66> "mysql-bin.000017", log_pos=633)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/plugin/semisync/semisync_master_plugin.cc:62
#3  0x00000000014b380b in Binlog_storage_delegate::after_flush (this=0x2dc8ee0 <delegates_init()::storage_mem>, thd=0x7f4018000e20,
    log_file=0x2debf02 <mysql_bin_log+66> "mysql-bin.000017", log_pos=633) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/rpl_handler.cc:574
#4  0x0000000001848716 in MYSQL_BIN_LOG::ordered_commit (this=0x2debec0 <mysql_bin_log>, thd=0x7f4018000e20, all=false, skip_commit=false)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/binlog.cc:9269
#5  0x0000000001846d22 in MYSQL_BIN_LOG::commit (this=0x2debec0 <mysql_bin_log>, thd=0x7f4018000e20, all=false)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/binlog.cc:8499
#6  0x0000000000f4ff78 in ha_commit_trans (thd=0x7f4018000e20, all=false, ignore_global_read_lock=false)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/handler.cc:1796
#7  0x0000000001691589 in trans_commit_stmt (thd=0x7f4018000e20) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/transaction.cc:458
#8  0x0000000001592ea5 in mysql_execute_command (thd=0x7f4018000e20, first_level=true) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5007
#9  0x00000000015943b9 in mysql_parse (thd=0x7f4018000e20, parser_state=0x7f407c2fb670) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:5577
#10 0x0000000001588d5e in dispatch_command (thd=0x7f4018000e20, com_data=0x7f407c2fbde0, command=COM_QUERY)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1461
#11 0x0000000001587b61 in do_command (thd=0x7f4018000e20) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#12 0x00000000016be3c0 in handle_connection (arg=0x4a242c0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#13 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4a7fb80) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#14 0x00007f4088fc06db in start_thread (arg=0x7f407c2fc700) at pthread_create.c:463
#15 0x00007f408795971f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```
```
第二行日志: 
 
#0  ActiveTranx::is_tranx_end_pos (this=0x4954540, log_file_name=0x7f407c33c1c2 "mysql-bin.000017", log_file_pos=633)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/plugin/semisync/semisync_master.cc:205
#1  0x00007f407c38ea84 in ReplSemiSyncMaster::updateSyncHeader (this=0x7f407c59b600 <repl_semisync>, packet=0x7f40202bf580 "", log_file_name=0x7f407c33c1c2 "mysql-bin.000017",
    log_file_pos=633, server_id=18722) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/plugin/semisync/semisync_master.cc:1108
#2  0x00007f407c390b56 in repl_semi_before_send_event (param=0x7f407c33b920, packet=0x7f40202bf580 "", len=34, log_file=0x7f407c33c1c2 "mysql-bin.000017", log_pos=633)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/plugin/semisync/semisync_master_plugin.cc:198
#3  0x00000000014b503e in Binlog_transmit_delegate::before_send_event (this=0x2dc9060 <delegates_init()::transmit_mem>, thd=0x7f402004b4d0, flags=0, packet=0x7f402004cf48,
    log_file=0x7f407c33c1c0 "./mysql-bin.000017", log_pos=633) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/rpl_handler.cc:795
#4  0x00000000018687b1 in Binlog_sender::before_send_hook (this=0x7f407c33c190, log_file=0x7f407c33c1c0 "./mysql-bin.000017", log_pos=633)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/rpl_binlog_sender.cc:1219
#5  0x0000000001865d57 in Binlog_sender::send_events (this=0x7f407c33c190, log_cache=0x7f407c33bf60, end_pos=633)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/rpl_binlog_sender.cc:444
#6  0x000000000186595d in Binlog_sender::send_binlog (this=0x7f407c33c190, log_cache=0x7f407c33bf60, start_pos=123)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/rpl_binlog_sender.cc:348
#7  0x0000000001865289 in Binlog_sender::run (this=0x7f407c33c190) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/rpl_binlog_sender.cc:225
#8  0x00000000018631da in mysql_binlog_send (thd=0x7f402004b4d0, log_ident=0x7f407c33cb80 "", pos=4, slave_gtid_executed=0x7f407c33ceb0, flags=0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/rpl_master.cc:412
#9  0x0000000001863097 in com_binlog_dump_gtid (thd=0x7f402004b4d0, packet=0x7f40203022c1 "", packet_length=86)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/rpl_master.cc:396
#10 0x00000000015898ad in dispatch_command (thd=0x7f402004b4d0, com_data=0x7f407c33dde0, command=COM_BINLOG_DUMP_GTID)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:1677
#11 0x0000000001587b61 in do_command (thd=0x7f402004b4d0) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/sql_parse.cc:999
#12 0x00000000016be3c0 in handle_connection (arg=0x7f4020000bb0)
    at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/sql/conn_handler/connection_handler_per_thread.cc:300
#13 0x0000000001910aa4 in pfs_spawn_thread (arg=0x4a55630) at /export/home/pb2/build/sb_0-24964902-1505322971.88/mysql-5.7.20/storage/perfschema/pfs.cc:2190
#14 0x00007f4088fc06db in start_thread (arg=0x7f407c33e700) at pthread_create.c:463
#15 0x00007f408795971f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

通过gdb, 已验证group commit的SYNC阶段后, 只要更新了binlog end pos (update_binlog_end_pos), 那么复制线程就可以进行复制, 而不需等待commit阶段

重新整理流程: 

```
- binlog.after_flush
	- repl_semi_report_binlog_update
		- writeTranxInBinlog
			- insert_tranx_node: 第一行日志

- sync_binlog_file
 
- update_binlog_end_pos (之后就可以进行复制, 而不需等待COMMIT阶段)

- Binlog_sender::send_binlog
	- get_binlog_end_pos
	- Binlog_sender::send_events
		- read_event
		- repl_semi_before_send_event
			- updateSyncHeader
				- is_tranx_end_pos: 第二行日志
``` 

探索代码, 发现在 Binlog_sender::get_binlog_end_pos 中获取binlog end pos时, 可能存在等待

```
int Binlog_sender::wait_new_events(my_off_t log_pos)
{
  int ret= 0;
  PSI_stage_info old_stage;

  mysql_bin_log.lock_binlog_end_pos();
  m_thd->ENTER_COND(mysql_bin_log.get_log_cond(),
                    mysql_bin_log.get_binlog_end_pos_lock(),
                    &stage_master_has_sent_all_binlog_to_slave,
                    &old_stage);

  if (mysql_bin_log.get_binlog_end_pos() <= log_pos &&
      mysql_bin_log.is_active(m_linfo.log_file_name))
  {
    if (m_heartbeat_period)
      ret= wait_with_heartbeat(log_pos);
    else
      ret= wait_without_heartbeat();
  }

  mysql_bin_log.unlock_binlog_end_pos();
  m_thd->EXIT_COND(&old_stage);
  return ret;
}
``` 

wait_new_events 的等待, 主要是 等待 (log_cache中的位置 log_pos < mysql_bin_log中标记的end_pos)

也就是说 等待 log_cache中的内容 "完全" 出现在 binlog 中

log event从 log_cache 写入 binlog的过程, 重新整理流程: 

```
- MYSQL_BIN_LOG::write_event, 将 log event 写入线程的cache中
 
- MYSQL_BIN_LOG::flush_thread_caches, 将线程的 cache 写入全局的log_cache
 
(这中间, 会存在log_cache中的log_event并未出现在binlog的情况, wait_new_events也为此状况存在)
 
- flush_cache_to_file
	- 将log_cache的内容确认刷入binlog
 
- binlog.after_flush
	- repl_semi_report_binlog_update
		- writeTranxInBinlog
			- insert_tranx_node: 第一行日志

- sync_binlog_file
 
- update_binlog_end_pos (之后就可以进行复制, 而不需等待COMMIT阶段)

- Binlog_sender::send_binlog
	- get_binlog_end_pos
	- Binlog_sender::send_events
		- read_event
		- repl_semi_before_send_event
			- updateSyncHeader
				- is_tranx_end_pos: 第二行日志
``` 

再次分析trace日志, 发现如下模式: 

```
(正常日志)
---
 
2021-07-14T13:34:44.319508+08:00 1119047 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28578784) in entry(11121)
2021-07-14T13:34:44.319861+08:00 1120132 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28579711) in entry(7657)
2021-07-14T13:34:44.320385+08:00 1120165 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28580682) in entry(11817)
2021-07-14T13:34:44.320698+08:00 1118606 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28581653) in entry(177)
... (大量均匀的ActiveTranx:insert_tranx_node)
2021-07-14T13:34:44.876017+08:00 1119737 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 29268524) in entry(8525)
2021-07-14T13:34:44.876815+08:00 1120195 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 29269499) in entry(13353)
 
---
 
2021-07-14T13:34:44.890422+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28577878) in entry(8909)
2021-07-14T13:34:44.890440+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28577878) sync(0), repl(1)
2021-07-14T13:34:44.890459+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28577952) in entry(12945)
2021-07-14T13:34:44.890463+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28577952) sync(0), repl(1)
2021-07-14T13:34:44.890470+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28578284) in entry(3377)
2021-07-14T13:34:44.890474+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28578284) sync(0), repl(1)
2021-07-14T13:34:44.890481+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28578379) in entry(5615)
2021-07-14T13:34:44.890485+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28578379) sync(0), repl(1)
2021-07-14T13:34:44.890500+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28578753) in entry(2265)
2021-07-14T13:34:44.890504+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28578753) sync(0), repl(1)
2021-07-14T13:34:44.890515+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28578784) in entry(11121)
2021-07-14T13:34:44.890519+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28578784) sync(1), repl(1)
2021-07-14T13:34:44.890533+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28578849) in entry(6503)
2021-07-14T13:34:44.890537+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28578849) sync(0), repl(1)
... (大量重复)
``` 

再分析日志, 观察28577813的记录: 

```
2021-07-14T13:34:44.319134+08:00 1118887 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28577813) in entry(8359)
2021-07-14T13:34:44.319289+08:00 1118887 [Note] ReplSemiSyncMaster::commitTrx: wait pos (mysql-bin.105618, 28577813), repl(1)
2021-07-14T13:34:44.319317+08:00 1118887 [Note] ReplSemiSyncMaster::commitTrx: init wait position (mysql-bin.105618, 28577813),
2021-07-14T13:34:44.319328+08:00 1118887 [Note] ReplSemiSyncMaster::commitTrx: wait 300000 ms for binlog sent (mysql-bin.105618, 28577813)
 
...

2021-07-14T13:34:44.319489+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28577813) in entry(8359)
2021-07-14T13:34:44.319496+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28577813) sync(1), repl(1)
 
...
 
(大量的如下日志)
2021-07-14T13:34:44.319508+08:00 1119047 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28578784) in entry(11121)
2021-07-14T13:34:44.319861+08:00 1120132 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28579711) in entry(7657)
2021-07-14T13:34:44.320385+08:00 1120165 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28580682) in entry(11817)
2021-07-14T13:34:44.320698+08:00 1118606 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28581653) in entry(177)
2021-07-14T13:34:44.320859+08:00 1118847 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28582626) in entry(6733)
2021-07-14T13:34:44.321235+08:00 1120025 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28583599) in entry(11343)
2021-07-14T13:34:44.323238+08:00 1120088 [Note] ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28584524) in entry(11545)
...
 
(部分如下日志)
2021-07-14T13:34:44.890422+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28577878) in entry(8909)
2021-07-14T13:34:44.890440+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28577878) sync(0), repl(1)
2021-07-14T13:34:44.890459+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28577952) in entry(12945)
2021-07-14T13:34:44.890463+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28577952) sync(0), repl(1)
2021-07-14T13:34:44.890470+08:00 584542 [Note] ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 28578284) in entry(3377)
2021-07-14T13:34:44.890474+08:00 584542 [Note] ReplSemiSyncMaster::updateSyncHeader: server(859389176), (mysql-bin.105618, 28578284) sync(0), repl(1)
...
 

2021-07-14T13:34:44.890827+08:00 0 [Note] ReplSemiSyncMaster::reportReplyPacket: Got reply(mysql-bin.105618, 28577813) from server 859389176
2021-07-14T13:34:44.890890+08:00 0 [Note] ReplSemiSyncMaster::reportReplyBinlog: Got reply at (mysql-bin.105618, 28577813)
2021-07-14T13:34:44.890953+08:00 1118887 [Note] ReplSemiSyncMaster::commitTrx: Binlog reply is ahead (mysql-bin.105618, 28577813),
2021-07-14T13:34:44.890991+08:00 1118887 [Note] ActiveTranx::::clear_active_tranx_nodes: cleared 1 nodes back until pos (mysql-bin.105618, 28578784)
``` 

两个异常: 

  1. 28577813 从发送event前 (updateSyncHeader), 到收到回复 (reportReplyPacket), 花费了700ms
  2. 大量 ActiveTranx:insert_tranx_node: insert 堆在一起, 而不会触发复制线程发送event. 两种可能
     1. 复制线程在发送event时阻塞
     2. log_cache.log_pos 生产过快, get_binlog_end_pos 一直在等待退出条件 (从代码判断不存在此种状况)

# semi-sync TRACE日志整理

  - Flush 阶段
    - fetch
    - process
      - MYSQL_BIN_LOG::flush_thread_caches: 将 线程级 的binlog 写入 log_cache
    - flush cache to file
      - 将 log_cache 写入 file
    - after_flush 钩子
      - 日志: ActiveTranx:insert_tranx_node: insert (mysql-bin.105618, 28577813) in entry(8359)
  - Sync 阶段
    - fetch
    - sync binlog
    - update binlog end pos: 更新binlog子系统的最新位置, 触发复制
      - 复制
        - get_binlog_end_pos
          - 等待 log_cache.log_pos < binlog.end_pos
            - log_cache.log_pos 由 flush 阶段process更新
            - binlog.end_pos 由 sync 阶段update binlog end pos更新
        - while ( log_pos < end_pos)
          - read event
          - before send 钩子
            - 日志: ActiveTranx::is_tranx_end_pos: probe (mysql-bin.105618, 24849333) in entry(2919)
            - 日志: ReplSemiSyncMaster::updateSyncHeader: server(1378179454), (mysql-bin.000047, 219386) sync(1), repl(1)
          - send
          - after send 钩子
            - repl_semi_after_send_event
              - read slave reply
              - net flush
  - 半同步ACK接收线程
    - 日志: ReplSemiSyncMaster::reportReplyPacket: Got reply(mysql-bin.105618, 24848356) from server 859389176
    - 若ACK达到规定数量
      - 日志: ReplSemiSyncMaster::reportReplyBinlog: Got reply at (mysql-bin.105618, 24848356)
      - 日志: ReplSemiSyncMaster::reportReplyBinlog: signal all waiting threads
  - Commit 阶段
    - fetch
    - after_sync 钩子
    - process
    - after_commit 钩子
      - 日志: ReplSemiSyncMaster::commitTrx: wait pos (mysql-bin.105618, 28577813), repl(1)

      - 初始化 等待的文件名
        - 日志: ReplSemiSyncMaster::commitTrx: init wait position (mysql-bin.105618, 28577813),
      - 日志: ReplSemiSyncMaster::commitTrx: wait 300000 ms for binlog sent (mysql-bin.105618, 28577813)
      - 开始等待半同步的ACK
      - 结束等待
      - 日志: ReplSemiSyncMaster::commitTrx: Binlog reply is ahead (mysql-bin.105618, 24843623)
      - 等待当前的ACK的最后一个事务? : 
        - 没有其他正在进行半同步的事务: 
          - 日志: ActiveTranx::::clear_active_tranx_nodes: cleared all nodes
        - 有其他正在进行半同步的事务: 
          - 日志: ActiveTranx::::clear_active_tranx_nodes: cleared 1 nodes back until pos (mysql-bin.105618, 24852353)
