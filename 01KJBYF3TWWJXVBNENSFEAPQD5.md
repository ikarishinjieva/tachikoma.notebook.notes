---
title: 20211029 - MySQL crash 分析
confluence_page_id: 1572933
created_at: 2021-10-29T08:21:33+00:00
updated_at: 2021-11-01T06:14:10+00:00
---

# 崩溃日志

```
2021-10-26T14:06:03.987288Z 0 [System] [MY-010931] [Server] /mysql/bin/mysqld: ready for connections. Version: '8.0.23-commercial'  socket: '/mysql/mysql.sock'  port: 3306  MySQL Enterprise Server - Commercial.
2021-10-26T14:06:04.008203Z 8 [Warning] [MY-010055] [Server] IP address '10.15.80.132' could not be resolved: Name or service not known
2021-10-26T14:06:04.022812Z 9 [Warning] [MY-010055] [Server] IP address '10.15.80.133' could not be resolved: Name or service not known
2021-10-26T14:30:54.527845Z 796 [Warning] [MY-010055] [Server] IP address '10.15.236.199' could not be resolved: Name or service not known
2021-10-27T02:16:02.832522Z 22459 [Warning] [MY-010055] [Server] IP address '10.15.237.182' could not be resolved: Name or service not known
06:39:20 UTC - mysqld got signal 11 ;
Most likely, you have hit a bug, but this error can also be caused by malfunctioning hardware.
Thread pointer: 0x7f2f2519eb40
Attempting backtrace. You can use the following information to find out
where mysqld died. If you see no messages after this, something went
terribly wrong...
stack_bottom = 7f30a81f2d40 thread_stack 0x40000
/mysql/bin/mysqld(my_print_stacktrace(unsigned char const*, unsigned long)+0x2e) [0x2066f7e]
/mysql/bin/mysqld(handle_fatal_signal+0x323) [0x1032d13]
/lib64/libpthread.so.0(+0xf630) [0x7f30b5e28630]
/mysql/bin/mysqld(ha_innobase::index_read(unsigned char*, unsigned char const*, unsigned int, ha_rkey_function)+0x2f) [0x214618f]
/mysql/bin/mysqld(ha_innobase::index_first(unsigned char*)+0x30) [0x2123240]
/mysql/bin/mysqld(ha_innobase::rnd_next(unsigned char*)+0x2f) [0x21470cf]
/mysql/bin/mysqld(handler::ha_rnd_next(unsigned char*)+0x16c) [0x112c2bc]
/mysql/bin/mysqld(Materialized_cursor::fetch(unsigned long)+0x83) [0xeac5c3]
/mysql/bin/mysqld(mysqld_stmt_fetch(THD*, Prepared_statement*, unsigned long)+0x109) [0xf28649]
/mysql/bin/mysqld(dispatch_command(THD*, COM_DATA const*, enum_server_command)+0x20f0) [0xf03670]
/mysql/bin/mysqld(do_command(THD*)+0x174) [0xf04234]
/mysql/bin/mysqld() [0x1024718]
/mysql/bin/mysqld() [0x259743c]
/lib64/libpthread.so.0(+0x7ea5) [0x7f30b5e20ea5]
/lib64/libc.so.6(clone+0x6d) [0x7f30b3f0596d]

Trying to get some variables.
Some pointers may be invalid and cause the dump to abort.
Query (7f2f2414a4a0): select * from bus_deposit_info
Connection ID (thread ID): 30558
Status: NOT_KILLED

The manual page at http://dev.mysql.com/doc/mysql/en/crashing.html contains
information that should help you find out what is causing the crash.
2021-10-27T06:39:20.253329Z mysqld_safe Number of processes running now: 0
2021-10-27T06:39:20.259186Z mysqld_safe mysqld restarted
``` 

发现数据库版本为: MySQL 企业版 8.0.23

# 诊断

获取了现场的coredump, 通过gdb进行检查

崩溃线程堆栈: 

```
(gdb) bt
#0  0x00007fc2e5bb8aa1 in pthread_kill () from /lib64/libpthread.so.0
#1  0x0000000002066547 in my_write_core (sig=<optimized out>) at ../../mysqlcom-8.0.23/include/my_thread.h:88
#2  0x0000000001032d3d in handle_fatal_signal (sig=11) at ../../mysqlcom-8.0.23/sql/signal_handler.cc:171
#3  <signal handler called>
#4  0x000000000214618f in ha_innobase::index_read (this=0x7fc15c04ee88, buf=0x7fc15c02ade8 "\001\374 0022cccd2e6e8ddfc6b9ff6211b48483",
    key_ptr=0x0, key_len=0, find_flag=HA_READ_AFTER_KEY) at ../../../mysqlcom-8.0.23/storage/innobase/handler/ha_innodb.cc:9707
#5  0x0000000002123240 in ha_innobase::index_first (this=0x7fc15c04ee88, buf=0x7fc15c02ade8 "\001\374 0022cccd2e6e8ddfc6b9ff6211b48483")
    at ../../../mysqlcom-8.0.23/storage/innobase/handler/ha_innodb.cc:10149
#6  0x00000000021470cf in ha_innobase::rnd_next (this=0x7fc15c04ee88, buf=0x7fc15c02ade8 "\001\374 0022cccd2e6e8ddfc6b9ff6211b48483")
    at ../../../mysqlcom-8.0.23/storage/innobase/handler/ha_innodb.cc:10321
#7  0x000000000112c2bc in handler::ha_rnd_next (this=0x7fc15c04ee88, buf=0x7fc15c02ade8 "\001\374 0022cccd2e6e8ddfc6b9ff6211b48483")
    at ../../mysqlcom-8.0.23/sql/handler.cc:2980
#8  0x0000000000eac5c3 in Materialized_cursor::fetch (this=0x7fc15c00da00, num_rows=<optimized out>)
    at ../../mysqlcom-8.0.23/sql/sql_cursor.cc:411
#9  0x0000000000f28649 in mysqld_stmt_fetch (thd=thd@entry=0x7fc15c000c20, stmt=0x7fc15c0109c0, num_rows=3000)
    at ../../mysqlcom-8.0.23/sql/sql_prepare.cc:1972
#10 0x0000000000f03670 in dispatch_command (thd=0x7fc15c000c20, com_data=<optimized out>, command=COM_STMT_FETCH)
    at ../../mysqlcom-8.0.23/sql/sql_parse.cc:1751
#11 0x0000000000f04234 in do_command (thd=thd@entry=0x7fc15c000c20) at ../../mysqlcom-8.0.23/sql/sql_parse.cc:1320
#12 0x0000000001024718 in handle_connection (arg=arg@entry=0x5449f20)
    at ../../mysqlcom-8.0.23/sql/conn_handler/connection_handler_per_thread.cc:301
#13 0x000000000259743c in pfs_spawn_thread (arg=0x53c56d0) at ../../../mysqlcom-8.0.23/storage/perfschema/pfs.cc:2900
#14 0x00007fc2e5bb3ea5 in start_thread () from /lib64/libpthread.so.0
#15 0x00007fc2e3c9896d in sysctl () from /lib64/libc.so.6
#16 0x0000000000000000 in ?? ()
``` 
    
    
    ha_innobase::index_read 相关代码: 

![image2021-10-29 16:21:13.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-29%2016%3A21%3A13.png)

通过gdb检查coredump, 发现其中m_prebuilt 为空

检查 this 结构, 认为可能是内存泄漏污染了this结构, 导致执行报错

this结构为: 

```
type = class handler {
  protected:
    TABLE_SHARE *table_share;
    TABLE *table;
    Table_flags cached_table_flags;
    Table_flags estimation_rows_to_insert;
  public:
    handlerton *ht;
    uchar *ref;
    uchar *dup_ref;
    ha_statistics stats;
    range_seq_t mrr_iter;
    RANGE_SEQ_IF mrr_funcs;
    HANDLER_BUFFER *multi_range_buffer;
    uint ranges_in_seq;
    bool mrr_is_output_sorted;
    bool mrr_have_range;
    KEY_MULTI_RANGE mrr_cur_range;
  private:
    Record_buffer *m_record_buffer;
    key_range save_end_range;
    handler::enum_range_scan_direction range_scan_direction;
    int key_compare_result_on_equal;
    handler *m_primary_handler;
  protected:
    KEY_PART_INFO *range_key_part;
    bool eq_range;
    bool in_range_check_pushed_down;
  public:
    key_range *end_range;
    bool m_virt_gcol_in_end_range;
    uint errkey;
    uint key_used_on_scan;
    uint active_index;
    uint ref_length;
    FT_INFO *ft_handler;
    enum : unsigned int {handler::NONE, handler::INDEX, handler::RND, handler::SAMPLING} inited;
    bool implicit_emptied;
    const Item *pushed_cond;
    Item *pushed_idx_cond;
    uint pushed_idx_cond_keyno;
    Table_flags next_insert_id;
    Table_flags insert_id_for_cur_row;
    Discrete_interval auto_inc_interval_for_cur_row;
    uint auto_inc_intervals_count;
    PSI_table *m_psi;
    std::mt19937 m_random_number_engine;
    double m_sampling_percentage;
  private:
    handler::batch_mode_t m_psi_batch_mode;
    Table_flags m_psi_numrows;
    PSI_table_locker *m_psi_locker;
    PSI_table_locker_state m_psi_locker_state;
    int m_lock_type;
    Handler_share **ha_share;
    bool m_update_generated_read_fields;
    Unique_on_insert *m_unique;

---

type = class ha_innobase : public handler {
  protected:
    DsMrr_impl m_ds_mrr;
    row_prebuilt_t *m_prebuilt;
    THD *m_user_thd;
    INNOBASE_SHARE *m_share;
    uchar *m_upd_buf;
    ulint m_upd_buf_size;
    handler::Table_flags m_int_table_flags;
    bool m_start_of_scan;
    uint m_last_match_mode;
    ulint m_stored_select_lock_type;
    bool m_mysql_has_locked;
``` 

检查各个指针, 内存泄漏会从异常指针位置开始. 认为可能从m_sampling_percentage附近开始泄露

检查相关内存: 

![image2021-10-30 11:41:39.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-30%2011%3A41%3A39.png)

内存呈现规律, 像是指针数组

向前检查内存, 认为可能从这些地址泄露: 

![image2021-10-30 11:46:49.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-30%2011%3A46%3A49.png)

该地址段为 ha_innobase 的 m_random_number_engine, 该结构长5000, 地址区间为: 0x7fc15c04f090 + 5000 = 0x7FC15C050418

此处比较可疑, 是指向 地址的指针: 

![image2021-10-30 23:30:12.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-30%2023%3A30%3A12.png)

# 内存特征-1

寻找其他内存特征, 发现: 

![image2021-10-31 0:23:52.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-31%200%3A23%3A52.png)

在正常MySQL中, 其特征为: 

![image2021-10-31 0:24:11.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-31%200%3A24%3A11.png)

其值含义为:

```
#define MEM_BLOCK_MAGIC_N 0x445566778899AABB
#define MEM_FREED_BLOCK_MAGIC_N 0xBBAA998877665544
``` 

# 内存特征-2

![image2021-10-31 0:37:32.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-31%200%3A37%3A32.png)

在正常MySQL中: 

![image2021-10-31 0:38:30.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-31%200%3A38%3A30.png)

在正常数据库中, 追踪这个标志的来源

在正常数据库中, 计算第一个出现的0x02826660 距离 ha_innobase 的偏移, 然后在此处进行 gdb的watch, 观察到其变更这个地址的堆栈为: 

```
(gdb) bt
#0  0x0000000001130d70 in Field::Field (this=0x7f89cc0176c8) at ../../mysql-8.0.25/sql/field.h:2969
#1  Field_temporal::Field_temporal (this=0x7f89cc0176c8) at ../../mysql-8.0.25/sql/field.h:2535
#2  Field_temporal_with_date::Field_temporal_with_date (this=0x7f89cc0176c8) at ../../mysql-8.0.25/sql/field.h:2750
#3  Field_temporal_with_date_and_time::Field_temporal_with_date_and_time (this=0x7f89cc0176c8) at ../../mysql-8.0.25/sql/field.h:2817
#4  Field_temporal_with_date_and_timef::Field_temporal_with_date_and_timef (this=0x7f89cc0176c8) at ../../mysql-8.0.25/sql/field.h:2867
#5  Field_timestampf::Field_timestampf (this=0x7f89cc0176c8) at ../../mysql-8.0.25/sql/field.h:2969
#6  Field_timestampf::clone (this=0x7f8a040ac678, mem_root=<optimized out>) at ../../mysql-8.0.25/sql/field.h:3001
#7  0x0000000001024fb4 in open_table_from_share(THD*, TABLE_SHARE*, char const*, unsigned int, unsigned int, unsigned int, TABLE*, bool, dd::Table const*) ()
    at ../../mysql-8.0.25/sql/table.cc:2981
#8  0x0000000000e9d0af in open_table(THD*, TABLE_LIST*, Open_table_context*) () at ../../mysql-8.0.25/sql/sql_base.cc:3364
#9  0x0000000000e9e6df in open_and_process_table (ot_ctx=0x7f8a40496950, has_prelocking_list=false, prelocking_strategy=0x7f8a404969e8,
    counter=0x7f8a404969dc, tables=0x7f89cc00c6e8, lex=<optimized out>, thd=0x7f89cc000e80) at ../../mysql-8.0.25/sql/sql_base.cc:5027
#10 open_tables (thd=0x7f89cc000e80, start=start@entry=0x7f8a404969e0, counter=counter@entry=0x7f8a404969dc, flags=flags@entry=18434,
    prelocking_strategy=prelocking_strategy@entry=0x7f8a404969e8) at ../../mysql-8.0.25/sql/sql_base.cc:5819
#11 0x0000000002051856 in open_tables (flags=18434, counter=0x7f8a404969dc, tables=0x7f8a404969e0, thd=<optimized out>)
    at ../../mysql-8.0.25/sql/sql_base.h:453
#12 dd::Open_dictionary_tables_ctx::open_tables (this=this@entry=0x7f8a40496a50) at ../../mysql-8.0.25/sql/dd/impl/transaction_impl.cc:107
#13 0x0000000001de98bc in dd::cache::Storage_adapter::get<dd::Item_name_key, dd::Schema> (thd=thd@entry=0x7f89cc000e80, key=...,
    isolation=isolation@entry=ISO_READ_COMMITTED, bypass_core_registry=bypass_core_registry@entry=false, object=object@entry=0x7f8a40496b18)
    at ../../mysql-8.0.25/sql/dd/impl/transaction_impl.h:90
#14 0x0000000001dddf3f in dd::cache::Shared_dictionary_cache::get_uncached<dd::Item_name_key, dd::Schema> (
    this=this@entry=0x3e081c0 <dd::cache::Shared_dictionary_cache::instance()::s_cache>, thd=thd@entry=0x7f89cc000e80, key=...,
    isolation=isolation@entry=ISO_READ_COMMITTED, object=object@entry=0x7f8a40496b18) at ../../mysql-8.0.25/sql/dd/impl/cache/shared_dictionary_cache.cc:109
#15 0x0000000001dddfa7 in dd::cache::Shared_dictionary_cache::get<dd::Item_name_key, dd::Schema> (
    this=0x3e081c0 <dd::cache::Shared_dictionary_cache::instance()::s_cache>, thd=0x7f89cc000e80, key=..., element=element@entry=0x7f8a40496b98)
    at ../../mysql-8.0.25/sql/dd/impl/cache/shared_dictionary_cache.h:174
#16 0x0000000001da2ff2 in dd::cache::Dictionary_client::acquire<dd::Item_name_key, dd::Schema> (this=this@entry=0x7f89cc0045e0, key=...,
    object=object@entry=0x7f8a40496c08, local_committed=local_committed@entry=0x7f8a40496c06, local_uncommitted=local_uncommitted@entry=0x7f8a40496c07)
    at ../../mysql-8.0.25/sql/dd/impl/object_key.h:44
#17 0x0000000001da3180 in dd::cache::Dictionary_client::acquire<dd::Schema> (this=this@entry=0x7f89cc0045e0, object_name="test",
    object=object@entry=0x7f8a40496d00) at ../../mysql-8.0.25/sql/dd/impl/cache/dictionary_client.cc:820
#18 0x0000000000eda222 in mysql_change_db(THD*, MYSQL_LEX_CSTRING const&, bool) () at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/unique_ptr.h:342
#19 0x0000000000f2e1db in dispatch_command(THD*, COM_DATA const*, enum_server_command) () at ../../mysql-8.0.25/sql/sql_parse.cc:1655
#20 0x0000000000f2f334 in do_command (thd=thd@entry=0x7f89cc000e80) at ../../mysql-8.0.25/sql/sql_parse.cc:1320
#21 0x0000000001052f98 in handle_connection (arg=arg@entry=0x6f4c680) at ../../mysql-8.0.25/sql/conn_handler/connection_handler_per_thread.cc:301
#22 0x00000000025aef7c in pfs_spawn_thread (arg=0x6fe4030) at ../../../mysql-8.0.25/storage/perfschema/pfs.cc:2898
#23 0x00007f8a510326db in start_thread (arg=0x7f8a40498700) at pthread_create.c:463
#24 0x00007f8a4eff871f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

可以得知: 

0x02826660 是 LEX_CSTRING 的空字符串的地址

```
constexpr const LEX_CSTRING EMPTY_CSTR = {"", 0};
``` 

Field的结构如下: 

```
(gdb) ptype this
type = class Field {
  protected:
    uchar *ptr;
  private:
    dd::Column::enum_hidden_type m_hidden;
    uchar *m_null_ptr;
    bool m_is_tmp_nullable;
    bool m_is_tmp_null;
    enum_check_fields m_check_for_truncated_fields_saved;
  protected:
    static uchar dummy_null_buffer;
  public:
    TABLE *table;
    const char *orig_db_name;
    const char *orig_table_name;
    const char **table_name;
    const char *field_name;
    LEX_CSTRING comment;
    Key_map key_start;
    Key_map part_of_key;
    Key_map part_of_prefixkey;
    Key_map part_of_sortkey;
    Key_map part_of_key_not_extended;
    static const size_t MAX_VARCHAR_WIDTH;
    static const size_t MAX_TINY_BLOB_WIDTH;
    static const size_t MAX_SHORT_BLOB_WIDTH;
    static const size_t MAX_MEDIUM_BLOB_WIDTH;
    static const size_t MAX_LONG_BLOB_WIDTH;
    uint32 field_length;
  private:
    uint32 flags;
    uint16 m_field_index;
  public:
    uchar null_bit;
    uchar auto_flags;
    bool is_created_from_null_item;
    bool m_indexed;
    LEX_CSTRING m_engine_attribute;
    LEX_CSTRING m_secondary_engine_attribute;
  private:
    unsigned int m_warnings_pushed;
  public:
---Type <return> to continue, or q <return> to quit---
    Value_generator *gcol_info;
    bool stored_in_db;
    Value_generator *m_default_val_expr;
``` 

其中 comment, m_engine_attribute, m_secondary_engine_attribute 涉及到 LEX_CSTRING, 偏移分别为: 

comment: 0x7f76dc017718 - 0x7f76dc0176c8 = 0x50

m_engine_attribute: 0x7f76dc017760 - 0x7f76dc0176c8 = 0x98

m_secondary_engine_attribute: 0x7f76dc017770 - 0x7f76dc0176c8 = 0xA8

以此算的, 在出问题的内存中

![image](http://8.134.54.170:8330/download/attachments/1572933/image2021-10-31%200%3A37%3A32.png?version=1&modificationDate=1635612032002&api=v2)

Field结构的起始位置为 0x7fc15c050100 - 0x98 = 0x7fc15c050068

![image2021-10-31 15:49:51.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-31%2015%3A49%3A51.png)

计算其comment的内容: 

![image2021-10-31 15:51:11.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-31%2015%3A51%3A11.png)

已知 Field是从MEM_ROOT中分配出来. 故障现象是 Field 从已经分配过的内存中再次分配, 导致内存数据不对影响cursor运行, 需要追查MEM_ROOT的分配原则

研究MySQL malloc的内存分布, 在my_malloc.cc中发现标识符 1234 = 0x4d2: 

```
#define MAGIC 1234
``` 

![image2021-10-31 18:40:18.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-31%2018%3A40%3A18.png)

my_memory_header 预留长度为32位, 但实际长度只有24位 ?? 

其内容为: 

![image2021-10-31 18:41:21.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-31%2018%3A41%3A21.png)

以下红色部分为 MEM_ROOT::Block, 参考: MEM_ROOT::AllocBlock

![image2021-10-31 18:50:32.png](/assets/01KJBYF3TWWJXVBNENSFEAPQD5/image2021-10-31%2018%3A50%3A32.png)

考虑 innobase结构体被异常释放? 否则alloc不能将其中空间分配出去

# 结论

用户直接升级至8.0.27, 现象消失. 查询release note, 认为命中了如下bug: 

<https://github.com/mysql/mysql-server/commit/627deede8ab0dc6ebde66eef5d868f73cfb5a0af>

```
Bug#32416811: Handler.ha_rnd_next failure
This failure may occur when we have a cursor created inside a
stored procedure and need to recreate the temporary table behind
the cursor because the storage limitation of the primary temporary
table is exceeded. In that case, the code that populates the temporary
table will recreate the temporary table in another storage engine, move
the contents of the primary table to the new one, and insert the
remaining part of the query result to this new table.

However, the handler for the new table is created on the current
THD's mem_root, which may not provide the storage lifetime expected
by cursors. Cursor objects exist over several roundtrips and we need
to enforce proper lifetime for them. For the primary temporay table,
this is ensured by using a special mem_root created in the cursor object.
However, we have no control over when the new temporary table may be
created, thus we may use the wrong mem_root for the allocation.

The fix is to save the memory allocator used when instantiating the
primary temporary table, and ensure to use the same allocator when
recreating the table with a different storage engine.
The pointer to the memory allocator is stored in the TABLE_SHARE object
and is only used for internal temporary tables.

Reviewed by: Sergey Glukhov <sergey.glukhov@oracle.com>

``` 

原理: 

当 存储过程中的 innodb cursor, 需要转换 内部临时表 (内存临时表 -> 文件临时表) 时, 文件临时表的mem_root设置错误, (错误地设置为 THD的mem_root).
