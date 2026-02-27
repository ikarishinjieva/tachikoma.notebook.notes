---
title: 20220628 - MySQL crash (bpage->buf_fix_count == 0)
confluence_page_id: 1933371
created_at: 2022-06-28T13:01:24+00:00
updated_at: 2022-07-05T06:41:09+00:00
---

# 问题

<https://support.actionsky.com/service_desk/browse/SHAI-6632>

背景：客户生产环境一主一从mysql架构，版本8.0.20，系统OpenEuler 主库22:35分 mysql crash，夜间无相关变更操作，业务操作等，无相关监控信息，无coredump文件，日志内有打印崩溃堆栈

Mysql-err日志如下(部分)：

2022-04-29T01:57:02.171635Z 1250605 [Warning] [MY-010055] [Server] IP address '***' could not be resolved: Name or service not known  
2022-06-10T14:35:43.044115Z 1419917 [ERROR] [MY-013183] [InnoDB] Assertion failure: [buf0lru.cc](<http://buf0lru.cc>):1997:bpage->buf_fix_count == 0 thread 28**58673012*44  
InnoDB: We intentionally generate a memory trap.  
InnoDB: Submit a detailed bug report to [http://bugs.mysql.com](<http://bugs.mysql.com/> "下列链接").  
InnoDB: If you get repeated assertion failures or crashes, even  
InnoDB: immediately after the mysqld startup, there may be  
InnoDB: corruption in the InnoDB tablespace. Please refer to  
InnoDB: <http://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html>  
InnoDB: about forcing recovery.  
Attempting backtrace. You can use the following information to find out  
where mysqld died. If you see no messages after this, something went  
/usr/sbin/mysqld(handle_fatal_signal+0x294) [0xee153c]  
[linux-vdso.so](<http://linux-vdso.so>).1(__kernel_rt_sigreturn+0) [0xfffc611407c0]  
/lib64/[libc.so](<http://libc.so>).6(gsignal+0xb0) [0xfffc607b6660]  
/lib64/[libc.so](<http://libc.so>).6(abort+0x154) [0xfffc607b7a0c]  
/usr/sbin/mysqld(ut_dbg_assertion_failed(char const*, char const*, unsigned long)+0x258) [0x1f64ef0]  
/usr/sbin/mysqld() [0x1feb240]  
/usr/sbin/mysqld(buf_LRU_free_page(buf_page_t*, bool)+0xcb4) [0x1fee084]  
/usr/sbin/mysqld(buf_LRU_scan_and_free_block(buf_pool_t*, bool)+0x850) [0x1feee90]  
/usr/sbin/mysqld(buf_LRU_get_free_block(buf_pool_t*)+0x4c4) [0x1fef684]  
/usr/sbin/mysqld(buf_page_init_for_read(dberr_t*, unsigned long, page_id_t const&, page_size_t const&, unsigned long)+0xf8) [0x1fc25d8]  
/usr/sbin/mysqld(buf_read_page_low(dberr_t*, bool, unsigned long, unsigned long, page_id_t const&, page_size_t const&, bool)+0x90) [0x1ff0798]  
/usr/sbin/mysqld(buf_read_page(page_id_t const&, page_size_t const&)+0x4c) [0x1ff0fb4]  
/usr/sbin/mysqld(Buf_fetch::read_page()+0x38) [0x1fc2c10]  
/usr/sbin/mysqld(Buf_fetch_normal::get(buf_block_t*&)+0x1c) [0x1fc2dcc]  
/usr/sbin/mysqld(Buf_fetch::single_page()+0x7c) [0x1fc2edc]  
/usr/sbin/mysqld(buf_page_get_gen(page_id_t const&, page_size_t const&, unsigned long, buf_block_t*, Page_fetch, char const*, unsigned long, mtr_t*, bool)+0x17c) [0x1fc4174]  
/usr/sbin/mysqld(btr_pcur_t::move_to_next_page(mtr_t*)+0xb4) [0x1fac614]  
/usr/sbin/mysqld(row_search_mvcc(unsigned char*, page_cur_mode_t, row_prebuilt_t*, unsigned long, unsigned long)+0xd2c) [0x1ee179c]  
/usr/sbin/mysqld(ha_innobase::general_fetch(unsigned char*, unsigned int, unsigned int)+0x184) [0x1d81d5c]  
/usr/sbin/mysqld(handler::ha_rnd_next(unsigned char*)+0x6c) [0xfc9cf4]  
/usr/sbin/mysqld(TableScanIterator::Read()+0x20) [0xd43d38]  
/usr/sbin/mysqld(HashJoinIterator::ReadRowFromProbeIterator()+0x20) [0xfdc5e8]  
/usr/sbin/mysqld(HashJoinIterator::Read()+0xb0) [0xfdc918]  
/usr/sbin/mysqld(MaterializeIterator::MaterializeQueryBlock(MaterializeIterator::QueryBlock const&, unsigned long long*)+0xd0) [0xf87728]  
/usr/sbin/mysqld(MaterializeIterator::Init()+0x244) [0xf87f5c]  
/usr/sbin/mysqld(filesort(THD*, Filesort*, RowIterator*, Filesort_info*, Sort_result*, unsigned long long*)+0x168) [0xfb91d0]  
/usr/sbin/mysqld(SortingIterator::DoSort()+0x84) [0xd55a64]  
/usr/sbin/mysqld(SortingIterator::Init()+0x1c) [0xd55afc]  
/usr/sbin/mysqld(SELECT_LEX_UNIT::ExecuteIteratorQuery(THD*)+0x1b4) [0xe7f774]  
/usr/sbin/mysqld(SELECT_LEX_UNIT::execute(THD*)+0x30) [0xe7fb78]  
/usr/sbin/mysqld(Sql_cmd_dml::execute_inner(THD*)+0x288) [0xe22b98]  
/usr/sbin/mysqld(Sql_cmd_dml::execute(THD*)+0x3b0) [0xe2b0c8]  
/usr/sbin/mysqld(mysql_execute_command(THD*, bool)+0xb28) [0xdddd90]  
/usr/sbin/mysqld(mysql_parse(THD*, Parser_state*)+0x30c) [0xde18c4]  
/usr/sbin/mysqld(dispatch_command(THD*, COM_DATA const*, enum_server_command)+0x150c) [0xde3154]  
/usr/sbin/mysqld(do_command(THD*)+0x15c) [0xde3d2c]  
/usr/sbin/mysqld() [0xed47e8]  
/usr/sbin/mysqld() [0x219a3cc]  
/lib64/[libpthread.so](<http://libpthread.so>).0(+0x88cc) [0xfffc610f88cc]  
/lib64/[libc.so](<http://libc.so>).6(+0xd954c) [0xfffc6085954c]

Trying to get some variables.  
Some pointers may be invalid and cause the dump to abort.  
Query (fffb492383c8): select [t.id](<http://t.id>),t.last_step_id,t.queue_id,t.queue_type,t.priority,t.execute_count,t.work_status,t.order_id,t.work_id,t.project_id,t.app_id,t.region_id,t.ipAddress,t.service_type,t.work_type,t.step,t.status,t.subscription_info,t.data_info,t.extend_info,t.start_date,t.end_date,t.update_date,t.create_date,t.operation_type,t.vm_res_id from order_queue_info t left join implementation_work iw on t.work_id = iw.work_id WHERE now() >= iw.implementaion_time and t.work_id = '20173' order by t.create_date asc  
Connection ID (thread ID): 1419917  
Status: NOT_KILLED

The manual page at <http://dev.mysql.com/doc/mysql/en/crashing.html> contains  
information that should help you find out what is causing the crash.  
2022-06-10T14:35:51.533051Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.20) starting as process 2900639  
2022-06-10T14:35:51.689400Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.  
2022-06-10T14:35:52.663532Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.  
2022-06-10T14:35:52.926607Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: '/var/run/mysqld/mysqlx.sock' bind-address: '::' port: 33060  
2022-06-10T14:35:54.725703Z 0 [System] [MY-010229] [Server] Starting XA crash recovery...  
2022-06-10T14:35:54.728857Z 0 [System] [MY-010232] [Server] XA crash recovery finished.  
2022-06-10T14:35:56.599918Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.

# 诊断

判断崩溃代码: 

```
buf_LRU_free_page(bpage) {
	block_mutex = buf_page_get_mutex(bpage);
	hash_lock = buf_page_hash_lock_get(buf_pool, bpage->id)
	ut_ad(mutex_own(block_mutex));
	...
	if (!buf_page_can_relocate(bpage)) {
		//如果buf_fix_count != 0, 退出
	} 
	...
	mutex_exit(block_mutex);
	rw_lock_x_lock(hash_lock);
	mutex_enter(block_mutex);
 
	if (!buf_page_can_relocate(bpage) || ...) {
		//如果buf_fix_count != 0, 退出
	}
	...
 
	buf_LRU_block_remove_hashed(bpage) {
		ut_a(bpage->buf_fix_count == 0); //此处 buf_fix_count != 0, 报错
		...
		rw_lock_x_unlock(hash_lock);
		mutex_exit(&((buf_block_t *)bpage)->mutex);
		...
	}
 
	...
}
``` 

可以判断, 在带着 hash_lock和block_mutex的过程中, buf_fix_count的值变了 (从0变成非0), 有两种可能: 

  - buf_fix_count 有非法的更新 (不带锁时进行的更新)
  - 内存污染

额外信息: 

  - buf_page_get_mutex (bpage)
    - 当bpage为 BUF_BLOCK_ZIP_PAGE/BUF_BLOCK_ZIP_DIRTY, 返回的是 对应 buffer pool的zip_mutex
    - 否则返回 bpage->mutex
  - buf_block_buf_fix_dec 不需要获取 mutex

先按照 buf_fix_count 有非法的更新 (不带锁时进行的更新) 进行诊断, 整理 buf_fix_count的变更点: 

  - buf_pool_watch_set
    - 已分析
  - Buf_fetch::zip_page_handler
    - 有 mutex的保护
  - buf_page_init
    - 有断言: 有mutex的保护
  - buf_page_init_for_read
    - 未分析
  - fil_buf_block_init
    - 只针对 新page, 可以忽略
  - buf_block_fix
    - buf_pool_watch_set
      - 有断言: hash lock保护
      - 有对 buf_pool->watch 的遍历处理, 其中涉及到 fix
        - 未分析
    - buf_page_get_zip
      - 已分析
    - Buf_fetch_normal::get
      - 有 hash lock 保护
    - Buf_fetch_other::get
      - 有 hash lock 保护
    - Buf_fetch::is_on_watch
      - 有 hash lock 保护
    - temp_space_page_handler
      - 有 mutex的保护
    - Buf_fetch::debug_check
      - debug用
    - buf_block_buf_fix_inc_func
      - buf_block_buf_fix_inc
        - PageBulk::release
          - BtrBulk 使用
            - fts_psort_insert: FTS parallel sort
            - row_merge_read_clustered_index: (Reads clustered index of the table and create temporary files

containing the index entries for the indexes to be built)

            - 未继续分析
        - btr_cur_optimistic_latch_leaves
          - buf_block_buf_fix_inc 有 mutex的保护
        - buf_page_get_zip
          - BUF_BLOCK_ZIP_PAGE
            - buf_block_fix 没有 mutex的保护
          - BUF_BLOCK_FILE_PAGE
            - buf_block_buf_fix_inc 有 mutex的保护
        - buf_page_optimistic_get
          - buf_block_buf_fix_inc 有 mutex的保护
        - buf_page_get_known_nowait
          - buf_block_buf_fix_inc 有 mutex的保护
        - buf_page_try_get_func
          - buf_block_buf_fix_inc 有 mutex的保护
        - buf_page_create
          - buf_block_buf_fix_inc 有 mutex的保护
        - rec_block_fix
          - BtrContext::check_redolog_bulk: lob?
          - BtrContext::restart_mtr_bulk: lob?

# 诊断 BtrContext::check_redolog_bulk

```
CREATE TABLE `a` (
  `a` int NOT NULL AUTO_INCREMENT,
  `c1` longblob,
  `c2` longblob,
  `c3` longblob,
  `c4` longblob,
  `c5` longblob,
  `c6` longblob,
  `c7` longblob,
  `c8` longblob,
  `c9` longblob,
  `c10` longblob,
  `c11` longblob,
  `c12` longblob,
  `c13` longblob,
  `c14` longblob,
  `c15` longblob,
  `c16` longblob,
  `c17` longblob,
  `c18` longblob
  PRIMARY KEY (`a`)
);
 
insert into test.a values(null,  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'),  
load_file('/tmp/gdb.txt'));
 
ALTER TABLE a add column (e int), ALGORITHM=inplace, LOCK=EXCLUSIVE;
``` 

可触发: 

```
#0  lob::BtrContext::check_redolog_bulk (this=0x7feef406def0)
    at /opt/mysql-server/storage/innobase/lob/lob0lob.cc:93
#1  0x0000561bd4b7006a in lob::BtrContext::check_redolog (this=0x7feef406def0)
    at /opt/mysql-server/storage/innobase/include/lob0lob.h:969
#2  0x0000561bd4b6bb99 in lob::btr_store_big_rec_extern_fields (trx=0x0, pcur=0x7feef406ef60,
    upd=0x0, offsets=0x7fee3aac6f50, big_rec_vec=0x7fee200000f0, btr_mtr=0x7fee3aac6870,
    op=lob::OPCODE_INSERT_BULK) at /opt/mysql-server/storage/innobase/lob/lob0lob.cc:473
#3  0x0000561bd4e97281 in PageBulk::storeExt (this=0x7fee38c721c0, big_rec=0x7fee200000f0,
    offsets=0x7fee3aac6f50) at /opt/mysql-server/storage/innobase/btr/btr0bulk.cc:576
#4  0x0000561bd4e95f01 in PageBulk::insert (this=0x7fee38c721c0, tuple=0x7fee3ab9d598,
    big_rec=0x7fee200000f0, rec_size=421)
    at /opt/mysql-server/storage/innobase/btr/btr0bulk.cc:193
#5  0x0000561bd4e97fb2 in BtrBulk::insert (this=0x7fee3aabb290, page_bulk=0x7fee38c721c0,
    tuple=0x7fee3ab9d598, big_rec=0x7fee200000f0, rec_size=421)
    at /opt/mysql-server/storage/innobase/btr/btr0bulk.cc:911
#6  0x0000561bd4e983fa in BtrBulk::insert (this=0x7fee3aabb290, tuple=0x7fee3ab9d598, level=0)
    at /opt/mysql-server/storage/innobase/btr/btr0bulk.cc:1000
#7  0x0000561bd4c8175c in BtrBulk::insert (this=0x7fee3aabb290, tuple=0x7fee3ab9d598)
    at /opt/mysql-server/storage/innobase/include/btr0bulk.h:318
#8  0x0000561bd4cb53d8 in row_merge_insert_index_tuples (trx=0x7feef6342058,
    index=0x7fee38ace660, old_table=0x7fee38aa9ab0, fd=-1, block=0x0, row_buf=0x7fee38d187f0,
    btr_bulk=0x7fee3aabb290, stage=0x0)
    at /opt/mysql-server/storage/innobase/row/row0merge.cc:3135
#9  0x0000561bd4cb2796 in row_merge_read_clustered_index (trx=0x7feef6342058,
    table=0x7fee3aaae390, old_table=0x7fee38aa9ab0, new_table=0x7fee38d155a0, online=false,
    index=0x7fee38d17d20, fts_sort_idx=0x0, psort_info=0x0, files=0x7fee38c42db0,
    key_numbers=0x7fee38d17d48, n_index=1, add_cols=0x7fee3aac55d8, add_v=0x0,
    col_map=0x7fee3aac58a0, add_autoinc=18446744073709551615, sequence=...,
    block=0x7feebc4fb000 "", skip_pk_sort=true, tmpfd=0x7feef40703e0, stage=0x7fee3aaa1e60,
    eval_table=0x7fee3aaae390) at /opt/mysql-server/storage/innobase/row/row0merge.cc:2224
#10 0x0000561bd4cb70f5 in row_merge_build_indexes (trx=0x7feef6342058, old_table=0x7fee38aa9ab0,
    new_table=0x7fee38d155a0, online=false, indexes=0x7fee38d17d20, key_numbers=0x7fee38d17d48,
    n_indexes=1, table=0x7fee3aaae390, add_cols=0x7fee3aac55d8, col_map=0x7fee3aac58a0,
    add_autoinc=18446744073709551615, sequence=..., skip_pk_sort=true, stage=0x7fee3aaa1e60,
    add_v=0x0, eval_table=0x7fee3aaae390)
    at /opt/mysql-server/storage/innobase/row/row0merge.cc:3743
#11 0x0000561bd4b2aa6e in ha_innobase::inplace_alter_table_impl<dd::Table> (this=0x7fee3aac9cc8,
    altered_table=0x7fee3aaae390, ha_alter_info=0x7feef4071350, old_dd_tab=0x7fee3aab5438,
    new_dd_tab=0x7fee3aa9e258) at /opt/mysql-server/storage/innobase/handler/handler0alter.cc:6038
#12 0x0000561bd4b04f12 in ha_innobase::inplace_alter_table (this=0x7fee3aac9cc8,
    altered_table=0x7fee3aaae390, ha_alter_info=0x7feef4071350, old_dd_tab=0x7fee3aab5438,
    new_dd_tab=0x7fee3aa9e258) at /opt/mysql-server/storage/innobase/handler/handler0alter.cc:1274
#13 0x0000561bd3402c90 in handler::ha_inplace_alter_table (this=0x7fee3aac9cc8,
    altered_table=0x7fee3aaae390, ha_alter_info=0x7feef4071350, old_table_def=0x7fee3aab5438,
    new_table_def=0x7fee3aa9e258) at /opt/mysql-server/sql/handler.h:5719
#14 0x0000561bd33e54ce in mysql_inplace_alter_table (thd=0x7fee38012a40, schema=...,
    new_schema=..., table_def=0x7fee3aab5438, altered_table_def=0x7fee3aa9e258,
    table_list=0x7fee38cf49d0, table=0x7fee3aabbe90, altered_table=0x7fee3aaae390,
    ha_alter_info=0x7feef4071350, inplace_supported=HA_ALTER_INPLACE_NO_LOCK_AFTER_PREPARE,
    alter_ctx=0x7feef4071da0, columns=..., fk_key_info=0x7fee38cd0260, fk_key_count=0,
    fk_invalidator=0x7feef4071290) at /opt/mysql-server/sql/sql_table.cc:12698
#15 0x0000561bd33f116b in mysql_alter_table (thd=0x7fee38012a40, new_db=0x7fee38cf4f88 "test",
    new_name=0x0, create_info=0x7feef40730e0, table_list=0x7fee38cf49d0,
    alter_info=0x7feef40731e0) at /opt/mysql-server/sql/sql_table.cc:16541
#16 0x0000561bd3997e59 in Sql_cmd_alter_table::execute (this=0x7fee38cf5088, thd=0x7fee38012a40)
    at /opt/mysql-server/sql/sql_alter.cc:349
#17 0x0000561bd3314d1d in mysql_execute_command (thd=0x7fee38012a40, first_level=true)
    at /opt/mysql-server/sql/sql_parse.cc:4489
#18 0x0000561bd331788c in mysql_parse (thd=0x7fee38012a40, parser_state=0x7feef4074cb0)
    at /opt/mysql-server/sql/sql_parse.cc:5306
#19 0x0000561bd330c634 in dispatch_command (thd=0x7fee38012a40, com_data=0x7feef4075ca0,
    command=COM_QUERY) at /opt/mysql-server/sql/sql_parse.cc:1776
#20 0x0000561bd330aad9 in do_command (thd=0x7fee38012a40)
    at /opt/mysql-server/sql/sql_parse.cc:1274
#21 0x0000561bd34ea2d6 in handle_connection (arg=0x561bda5b3990)
    at /opt/mysql-server/sql/conn_handler/connection_handler_per_thread.cc:302
#22 0x0000561bd51f3515 in pfs_spawn_thread (arg=0x561bda5c9cc0)
    at /opt/mysql-server/storage/perfschema/pfs.cc:2854
#23 0x00007fef0580e6db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#24 0x00007fef039c971f in clone () from /lib/x86_64-linux-gnu/libc.so.6
``` 

经确认, 此处的 m_block, 在 rec_block_fix 前, fix_count = 1, 已经不会从LRU中换出, 不符合触发条件: 

```
(gdb) bt
#0  lob::BtrContext::check_redolog_bulk (this=0x7fffe80a8ef0) at /opt/mysql-server/storage/innobase/lob/lob0lob.cc:93
...
 
(gdb) p *m_block
$2 = {page = {id = {m_space = 16, m_page_no = 5, m_fold = 16777237}, size = {m_physical = 16384, m_logical = 16384, m_is_compressed = 0}, buf_fix_count = 1, io_fix = BUF_IO_NONE, state = BUF_BLOCK_FILE_PAGE, flush_type = 0, buf_pool_index = 0, zip = {data = 0x0, m_start = 0, m_external = false, m_end = 0, m_nonempty = 0, n_blobs = 0, ssize = 0}, hash = 0x0, in_page_hash = true, in_zip_hash = false, list = {prev = 0x0, next = 0x0}, in_flush_list = false, in_free_list = false, newest_modification = 0, oldest_modification = 0, LRU = {prev = 0x7fffd3d96900, next = 0x7fffd3d93300}, in_LRU_list = true, old = 0, freed_page_clock = 753, access_time = 2451998747, file_page_was_freed = false, flush_observer = 0x0, m_dblwr_id = 0}, frame = 0x7fffd8c88000 "", lock = {<latch_t> = {_vptr.latch_t = 0x55555ce64820 <vtable for rw_lock_t+16>, m_id = LATCH_ID_BUF_BLOCK_LOCK, m_rw_lock = true, m_temp_fsp = false}, lock_word = 0, waiters = 0, recursive = {_M_base = {static _S_alignment = 1, _M_i = true}}, sx_recursive = 0, writer_is_wait_ex = false, writer_thread = {<std::__atomic_base<unsigned long>> = {static _S_alignment = 8, _M_i = 140737086428928}, <No data fields>}, event = 0x7fffdc3bdf70, wait_ex_event = 0x7fffdc3be010, cfile_name = 0x55555beaaaf8 "/opt/mysql-server/storage/innobase/buf/buf0buf.cc", last_s_file_name = 0x55555be59b26 "not yet reserved", last_x_file_name = 0x55555beaaaf8 "/opt/mysql-server/storage/innobase/buf/buf0buf.cc", cline = 778, is_block_lock = 1, last_s_line = 0, last_x_line = 4861, count_os_wait = 0, list = {prev = 0x7fffd3d967d0, next = 0x7fffd3d964d0}, pfs_psi = 0x0, static MAGIC_N = 22643, magic_n = 22643, debug_list = {count = 1, start = 0x7fff34c48680, end = 0x7fff34c48680, node = &rw_lock_debug_t::list, init = 51966}, level = SYNC_LEVEL_VARYING}, unzip_LRU = {prev = 0x0, next = 0x0}, in_unzip_LRU_list = false, in_withdraw_list = false, lock_hash_val = 5275, modify_clock = 1, n_hash_helps = 0, n_bytes = 0, n_fields = 1, left_side = true, n_pointers = 0, curr_n_fields = 0, curr_n_bytes = 0, curr_left_side = 0, index = 0x0, made_dirty_with_no_latch = false, debug_latch = {<latch_t> = {_vptr.latch_t = 0x55555ce64820 <vtable for rw_lock_t+16>, m_id = LATCH_ID_DICT_FOREIGN_ERR, m_rw_lock = true, m_temp_fsp = false}, lock_word = 536870911, waiters = 0, recursive = {_M_base = {static _S_alignment = 1, _M_i = false}}, sx_recursive = 0, writer_is_wait_ex = false, writer_thread = {<std::__atomic_base<unsigned long>> = {static _S_alignment = 8, _M_i = 0}, <No data fields>}, event = 0x7fffdc3be0b0, wait_ex_event = 0x7fffdc3be150, cfile_name = 0x55555beaaaf8 "/opt/mysql-server/storage/innobase/buf/buf0buf.cc", last_s_file_name = 0x55555beaaaf8 "/opt/mysql-server/storage/innobase/buf/buf0buf.cc", last_x_file_name = 0x55555be59b26 "not yet reserved", cline = 781, is_block_lock = 0, last_s_line = 4848, last_x_line = 0, count_os_wait = 0, list = {prev = 0x7fffd3d969a8, next = 0x7fffd3d966a8}, pfs_psi = 0x0, static MAGIC_N = 22643, magic_n = 22643, debug_list = {count = 1, start = 0x7fff34c487a0, end = 0x7fff34c487a0, node = &rw_lock_debug_t::list, init = 51966}, level = SYNC_NO_ORDER_CHECK}, mutex = {m_impl = {m_lock_word = {_M_base = {static _S_alignment = 1, _M_i = false}}, m_waiters = {_M_base = {static _S_alignment = 1, _M_i = false}}, m_event = 0x7fffdc3bded0, m_policy = {<MutexDebug<TTASEventMutex<BlockMutexPolicy> >> = {_vptr.MutexDebug = 0x55555ce64b80 <vtable for BlockMutexPolicy<TTASEventMutex<BlockMutexPolicy> >+16>, m_magic_n = 979585, m_context = {<latch_t> = {_vptr.latch_t = 0x55555ce62bf8 <vtable for MutexDebug<TTASEventMutex<BlockMutexPolicy> >::Context+16>, m_id = LATCH_ID_BUF_BLOCK_MUTEX, m_rw_lock = false, m_temp_fsp = false}, m_mutex = 0x0, m_filename = 0x0, m_line = 18446744073709551615, m_thread_id = 18446744073709551615}}, m_count = 0x7fffdc0014d0, m_id = LATCH_ID_BUF_BLOCK_MUTEX}}, m_ptr = 0x0}}
 
 
//buf_fix_count 已经为1
``` 

# 暂停

没有好的手段进行继续诊断, 等待coredump和下次重现
