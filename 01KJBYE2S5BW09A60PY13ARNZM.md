---
title: 20210730 - i_s.columns 排序不一致
confluence_page_id: 1343566
created_at: 2021-07-30T07:51:50+00:00
updated_at: 2021-08-02T04:16:26+00:00
---

# 现象

建表语句: 

```
CREATE TABLE `rsp_cfg_anlstrdfn` (
  `SBJCOD` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL,
  `ANLCOD` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL,
  `COLCOD` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL
)
``` 

i_s.columns 查询语句: 

```
select * from information_schema.COLUMNS where table_name = 'rsp_cfg_anlstrdfn'\G
``` 

发现: 

MySQL 8.0.19 的输出: 

(dbdeployer deploy single 8.0.19 --master --gtid)

![image2021-7-30 15:41:34.png](/assets/01KJBYE2S5BW09A60PY13ARNZM/image2021-7-30%2015%3A41%3A34.png)

调整lower_case_table_names=1后, 输出会按照 ORDINAL_POSITION 排序: 

(dbdeployer deploy single 8.0.19 --master --gtid -c lower_case_table_names=1 -i --lower_case_table_names=1)

![image2021-7-30 15:38:48.png](/assets/01KJBYE2S5BW09A60PY13ARNZM/image2021-7-30%2015%3A38%3A48.png)

而在另外的机器上, 即使 lower_case_table_names=1, 也仍然会按照 TABLE_NAME 排序

# 分析

i_s.columns是一个视图, 并没有特殊排序语句. 其中 涉及的mysql.columns是内部表.

select访问内部表的堆栈如下: 

```
#0  dict_table_t::first_index (this=0x7f1a940e2ca0)
    at ../../../mysql-8.0.25/storage/innobase/include/dict0mem.h:2066
#1  0x0000000004acab0a in row_search_mvcc (buf=0x7f1a940e1a78 "", mode=PAGE_CUR_G,
    prebuilt=0x7f1a940e6770, match_mode=0, direction=0)
    at ../../../mysql-8.0.25/storage/innobase/row/row0sel.cc:4835
#2  0x0000000004859ac8 in ha_innobase::index_read (this=0x7f1a940e03a8,
    buf=0x7f1a940e1a78 "", key_ptr=0x0, key_len=0, find_flag=HA_READ_AFTER_KEY)
    at ../../../mysql-8.0.25/storage/innobase/handler/ha_innodb.cc:9956
#3  0x000000000485abfd in ha_innobase::index_first (this=0x7f1a940e03a8,
    buf=0x7f1a940e1a78 "")
    at ../../../mysql-8.0.25/storage/innobase/handler/ha_innodb.cc:10305
#4  0x000000000351fb2b in handler::ha_index_first (this=0x7f1a940e03a8,
    buf=0x7f1a940e1a78 "") at ../../mysql-8.0.25/sql/handler.cc:3393
#5  0x0000000003063a6d in IndexScanIterator<false>::Read (this=0x7f1a9411c9f8)
    at ../../mysql-8.0.25/sql/records.cc:107
#6  0x00000000038471fe in NestedLoopIterator::Read (this=0x7f1a9411ca78)
    at ../../mysql-8.0.25/sql/composite_iterators.cc:431
#7  0x00000000038471fe in NestedLoopIterator::Read (this=0x7f1a9411cb08)
    at ../../mysql-8.0.25/sql/composite_iterators.cc:431
#8  0x00000000038471fe in NestedLoopIterator::Read (this=0x7f1a9411cb98)
    at ../../mysql-8.0.25/sql/composite_iterators.cc:431
#9  0x00000000038471fe in NestedLoopIterator::Read (this=0x7f1a9411cbf8)
    at ../../mysql-8.0.25/sql/composite_iterators.cc:431
#10 0x00000000038471fe in NestedLoopIterator::Read (this=0x7f1a9411cc58)
    at ../../mysql-8.0.25/sql/composite_iterators.cc:431
#11 0x00000000032d22e5 in Query_expression::ExecuteIteratorQuery (this=0x7f1a9400c0e8,
    thd=0x7f1a94000f00) at ../../mysql-8.0.25/sql/sql_union.cc:1230
#12 0x00000000032d2611 in Query_expression::execute (this=0x7f1a9400c0e8,
    thd=0x7f1a94000f00) at ../../mysql-8.0.25/sql/sql_union.cc:1283
#13 0x000000000322be9c in Sql_cmd_dml::execute_inner (this=0x7f1a9412bcd8,
    thd=0x7f1a94000f00) at ../../mysql-8.0.25/sql/sql_select.cc:791
#14 0x000000000322b418 in Sql_cmd_dml::execute (this=0x7f1a9412bcd8,
    thd=0x7f1a94000f00) at ../../mysql-8.0.25/sql/sql_select.cc:575
#15 0x00000000031b2496 in mysql_execute_command (thd=0x7f1a94000f00, first_level=true)
    at ../../mysql-8.0.25/sql/sql_parse.cc:4412
#16 0x00000000031b43b0 in dispatch_sql_command (thd=0x7f1a94000f00,
    parser_state=0x7f1b4c0aeb70) at ../../mysql-8.0.25/sql/sql_parse.cc:5000
#17 0x00000000031aa9d2 in dispatch_command (thd=0x7f1a94000f00,
    com_data=0x7f1b4c0afc20, command=COM_QUERY)
    at ../../mysql-8.0.25/sql/sql_parse.cc:1841
#18 0x00000000031a8e0c in do_command (thd=0x7f1a94000f00)
---Type <return> to continue, or q <return> to quit---
    at ../../mysql-8.0.25/sql/sql_parse.cc:1320
#19 0x000000000337b4a7 in handle_connection (arg=0x9a16410)
    at ../../mysql-8.0.25/sql/conn_handler/connection_handler_per_thread.cc:301
#20 0x0000000004f499d3 in pfs_spawn_thread (arg=0xb820270)
    at ../../../mysql-8.0.25/storage/perfschema/pfs.cc:2898
#21 0x00007f1b5eda86db in start_thread (arg=0x7f1b4c0b0700) at pthread_create.c:463
#22 0x00007f1b5cd6e71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

其从InnoDB表中直接读取数据, 没有特殊的排序操作.

那么应当是数据存入mysql.columns的过程进行了排序, 堆栈如下: (跟踪点名: before_insert_into_dd)

```
#0  ha_innobase::write_row (this=0x7f1a94060888,
    record=0x7f1a94061f58 "\370\377\377\377")
    at ../../../mysql-8.0.25/storage/innobase/handler/ha_innodb.cc:8698
#1  0x000000000352af94 in handler::ha_write_row (this=0x7f1a94060888,
    buf=0x7f1a94061f58 "\370\377\377\377") at ../../mysql-8.0.25/sql/handler.cc:7832
#2  0x00000000045fec6b in dd::Raw_new_record::insert (this=0x7f1a9419b9a0)
    at ../../mysql-8.0.25/sql/dd/impl/raw/raw_record.cc:311
#3  0x00000000046c091b in dd::Weak_object_impl_<true>::store (this=0x7f1a94113c70,
    otx=0x7f1b4c0ab6c0) at ../../mysql-8.0.25/sql/dd/impl/types/weak_object_impl.cc:129
#4  0x00000000045e747d in dd::cache::Storage_adapter::store<dd::Table> (
    thd=0x7f1a94000f00, object=0x7f1a94113f50)
    at ../../mysql-8.0.25/sql/dd/impl/cache/storage_adapter.cc:333
#5  0x00000000044ffcb4 in dd::cache::Dictionary_client::store<dd::Table> (
    this=0x7f1a94004780, object=0x7f1a94113f50)
    at ../../mysql-8.0.25/sql/dd/impl/cache/dictionary_client.cc:2492
#6  0x000000000325d8a6 in rea_create_base_table (thd=0x7f1a94000f00,
    path=0x7f1b4c0accc0 "./test/b22", sch_obj=..., db=0x7f1a94163670 "test",
    table_name=0x7f1a94162b60 "b22", create_info=0x7f1b4c0ad420, create_fields=...,
    keys=0, key_info=0x7f1a94165210, keys_onoff=Alter_info::ENABLE, fk_keys=0,
    fk_key_info=0x7f1a94165210, check_cons_spec=0x7f1b4c0ad370, file=0x7f1a94163b00,
    no_ha_table=false, do_not_store_in_dd=false, part_info=0x0,
    binlog_to_trx_cache=0x7f1b4c0ad0ef, table_def_ptr=0x7f1b4c0aced0,
    post_ddl_ht=0x7f1b4c0ad0e0) at ../../mysql-8.0.25/sql/sql_table.cc:1084
#7  0x00000000032714ad in create_table_impl (thd=0x7f1a94000f00, schema=...,
    db=0x7f1a94163670 "test", table_name=0x7f1a94162b60 "b22",
    error_table_name=0x7f1a94162b60 "b22", path=0x7f1b4c0accc0 "./test/b22",
    create_info=0x7f1b4c0ad420, alter_info=0x7f1b4c0ad2b0, internal_tmp_table=false,
    select_field_count=0, find_parent_keys=true, no_ha_table=false,
    do_not_store_in_dd=false, is_trans=0x7f1b4c0ad0ef, key_info=0x7f1b4c0acef0,
    key_count=0x7f1b4c0aceec, keys_onoff=Alter_info::ENABLE,
    fk_key_info=0x7f1b4c0acee0, fk_key_count=0x7f1b4c0acedc, existing_fk_info=0x0,
    existing_fk_count=0, existing_fk_table=0x0, fk_max_generated_name_number=0,
    table_def=0x7f1b4c0aced0, post_ddl_ht=0x7f1b4c0ad0e0)
    at ../../mysql-8.0.25/sql/sql_table.cc:8709
#8  0x00000000032720d1 in mysql_create_table_no_lock (thd=0x7f1a94000f00,
    db=0x7f1a94163670 "test", table_name=0x7f1a94162b60 "b22",
    create_info=0x7f1b4c0ad420, alter_info=0x7f1b4c0ad2b0, select_field_count=0,
    find_parent_keys=true, is_trans=0x7f1b4c0ad0ef, post_ddl_ht=0x7f1b4c0ad0e0)
    at ../../mysql-8.0.25/sql/sql_table.cc:8950
#9  0x00000000032756d5 in mysql_create_table (thd=0x7f1a94000f00,
    create_table=0x7f1a94163058, create_info=0x7f1b4c0ad420, alter_info=0x7f1b4c0ad2b0)
---Type <return> to continue, or q <return> to quit---
    at ../../mysql-8.0.25/sql/sql_table.cc:9811
#10 0x00000000037f0dbd in Sql_cmd_create_table::execute (this=0x7f1a94163790,
    thd=0x7f1a94000f00) at ../../mysql-8.0.25/sql/sql_cmd_ddl_table.cc:406
#11 0x00000000031af265 in mysql_execute_command (thd=0x7f1a94000f00, first_level=true)
    at ../../mysql-8.0.25/sql/sql_parse.cc:3426
#12 0x00000000031b43b0 in dispatch_sql_command (thd=0x7f1a94000f00,
    parser_state=0x7f1b4c0aeb70) at ../../mysql-8.0.25/sql/sql_parse.cc:5000
#13 0x00000000031aa9d2 in dispatch_command (thd=0x7f1a94000f00,
    com_data=0x7f1b4c0afc20, command=COM_QUERY)
    at ../../mysql-8.0.25/sql/sql_parse.cc:1841
#14 0x00000000031a8e0c in do_command (thd=0x7f1a94000f00)
    at ../../mysql-8.0.25/sql/sql_parse.cc:1320
#15 0x000000000337b4a7 in handle_connection (arg=0x9a16410)
    at ../../mysql-8.0.25/sql/conn_handler/connection_handler_per_thread.cc:301
#16 0x0000000004f499d3 in pfs_spawn_thread (arg=0xb820270)
    at ../../../mysql-8.0.25/storage/perfschema/pfs.cc:2898
#17 0x00007f1b5eda86db in start_thread (arg=0x7f1b4c0b0700) at pthread_create.c:463
#18 0x00007f1b5cd6e71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

create table过程中, 生成column元数据的堆栈如下: 

```
#0  dd::Raw_new_record::get_insert_id (this=0x7f1a9419b9a0)
    at ../../mysql-8.0.25/sql/dd/impl/raw/raw_record.cc:327
#1  0x00000000046802fe in dd::Entity_object_impl::set_primary_key_value (
    this=0x7f1a94141df0, r=...)
    at ../../mysql-8.0.25/sql/dd/impl/types/entity_object_impl.cc:56
#2  0x00000000046c0995 in dd::Weak_object_impl_<true>::store (this=0x7f1a94141df0,
    otx=0x7f1b4c0ab6c0) at ../../mysql-8.0.25/sql/dd/impl/types/weak_object_impl.cc:136
#3  0x000000000444bc83 in dd::Collection<dd::Column*>::store_items (
    this=0x7f1a94113d10, otx=0x7f1b4c0ab6c0)
    at ../../mysql-8.0.25/sql/dd/collection.cc:218
#4  0x0000000004670994 in dd::Abstract_table_impl::store_children (
    this=0x7f1a94113c70, otx=0x7f1b4c0ab6c0)
    at ../../mysql-8.0.25/sql/dd/impl/types/abstract_table_impl.cc:126
#5  0x00000000046a6f9e in dd::Table_impl::store_children (this=0x7f1a94113c70,
    otx=0x7f1b4c0ab6c0) at ../../mysql-8.0.25/sql/dd/impl/types/table_impl.cc:365
#6  0x00000000046c09cd in dd::Weak_object_impl_<true>::store (this=0x7f1a94113c70,
    otx=0x7f1b4c0ab6c0) at ../../mysql-8.0.25/sql/dd/impl/types/weak_object_impl.cc:152
#7  0x00000000045e747d in dd::cache::Storage_adapter::store<dd::Table> (
    thd=0x7f1a94000f00, object=0x7f1a94113f50)
    at ../../mysql-8.0.25/sql/dd/impl/cache/storage_adapter.cc:333
#8  0x00000000044ffcb4 in dd::cache::Dictionary_client::store<dd::Table> (
    this=0x7f1a94004780, object=0x7f1a94113f50)
    at ../../mysql-8.0.25/sql/dd/impl/cache/dictionary_client.cc:2492
#9  0x000000000325d8a6 in rea_create_base_table (thd=0x7f1a94000f00,
    path=0x7f1b4c0accc0 "./test/b22", sch_obj=..., db=0x7f1a94163670 "test",
    table_name=0x7f1a94162b60 "b22", create_info=0x7f1b4c0ad420, create_fields=...,
    keys=0, key_info=0x7f1a94165210, keys_onoff=Alter_info::ENABLE, fk_keys=0,
    fk_key_info=0x7f1a94165210, check_cons_spec=0x7f1b4c0ad370, file=0x7f1a94163b00,
    no_ha_table=false, do_not_store_in_dd=false, part_info=0x0,
    binlog_to_trx_cache=0x7f1b4c0ad0ef, table_def_ptr=0x7f1b4c0aced0,
    post_ddl_ht=0x7f1b4c0ad0e0) at ../../mysql-8.0.25/sql/sql_table.cc:1084
#10 0x00000000032714ad in create_table_impl (thd=0x7f1a94000f00, schema=...,
    db=0x7f1a94163670 "test", table_name=0x7f1a94162b60 "b22",
    error_table_name=0x7f1a94162b60 "b22", path=0x7f1b4c0accc0 "./test/b22",
    create_info=0x7f1b4c0ad420, alter_info=0x7f1b4c0ad2b0, internal_tmp_table=false,
    select_field_count=0, find_parent_keys=true, no_ha_table=false,
    do_not_store_in_dd=false, is_trans=0x7f1b4c0ad0ef, key_info=0x7f1b4c0acef0,
    key_count=0x7f1b4c0aceec, keys_onoff=Alter_info::ENABLE,
    fk_key_info=0x7f1b4c0acee0, fk_key_count=0x7f1b4c0acedc, existing_fk_info=0x0,
    existing_fk_count=0, existing_fk_table=0x0, fk_max_generated_name_number=0,
    table_def=0x7f1b4c0aced0, post_ddl_ht=0x7f1b4c0ad0e0)
---Type <return> to continue, or q <return> to quit---
    at ../../mysql-8.0.25/sql/sql_table.cc:8709
#11 0x00000000032720d1 in mysql_create_table_no_lock (thd=0x7f1a94000f00,
    db=0x7f1a94163670 "test", table_name=0x7f1a94162b60 "b22",
    create_info=0x7f1b4c0ad420, alter_info=0x7f1b4c0ad2b0, select_field_count=0,
    find_parent_keys=true, is_trans=0x7f1b4c0ad0ef, post_ddl_ht=0x7f1b4c0ad0e0)
    at ../../mysql-8.0.25/sql/sql_table.cc:8950
#12 0x00000000032756d5 in mysql_create_table (thd=0x7f1a94000f00,
    create_table=0x7f1a94163058, create_info=0x7f1b4c0ad420, alter_info=0x7f1b4c0ad2b0)
    at ../../mysql-8.0.25/sql/sql_table.cc:9811
#13 0x00000000037f0dbd in Sql_cmd_create_table::execute (this=0x7f1a94163790,
    thd=0x7f1a94000f00) at ../../mysql-8.0.25/sql/sql_cmd_ddl_table.cc:406
#14 0x00000000031af265 in mysql_execute_command (thd=0x7f1a94000f00, first_level=true)
    at ../../mysql-8.0.25/sql/sql_parse.cc:3426
#15 0x00000000031b43b0 in dispatch_sql_command (thd=0x7f1a94000f00,
    parser_state=0x7f1b4c0aeb70) at ../../mysql-8.0.25/sql/sql_parse.cc:5000
#16 0x00000000031aa9d2 in dispatch_command (thd=0x7f1a94000f00,
    com_data=0x7f1b4c0afc20, command=COM_QUERY)
    at ../../mysql-8.0.25/sql/sql_parse.cc:1841
#17 0x00000000031a8e0c in do_command (thd=0x7f1a94000f00)
    at ../../mysql-8.0.25/sql/sql_parse.cc:1320
#18 0x000000000337b4a7 in handle_connection (arg=0x9a16410)
    at ../../mysql-8.0.25/sql/conn_handler/connection_handler_per_thread.cc:301
#19 0x0000000004f499d3 in pfs_spawn_thread (arg=0xb820270)
    at ../../../mysql-8.0.25/storage/perfschema/pfs.cc:2898
#20 0x00007f1b5eda86db in start_thread (arg=0x7f1b4c0b0700) at pthread_create.c:463
#21 0x00007f1b5cd6e71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 
    
    
    DDL之后, 重新打开表, column会重新排序, 按照唯一键的指定排序:

```
#0  dd::tables::Columns::create_key_by_table_id (table_id=411) at ../../mysql-8.0.25/sql/dd/impl/tables/columns.cc:163
#1  0x0000000004670939 in dd::Abstract_table_impl::restore_children (this=0x7f1a94140260, otx=0x7f1b4c0a9970) at ../../mysql-8.0.25/sql/dd/impl/types/abstract_table_impl.cc:119
#2  0x00000000046a6bc1 in dd::Table_impl::restore_children (this=0x7f1a94140260, otx=0x7f1b4c0a9970) at ../../mysql-8.0.25/sql/dd/impl/types/table_impl.cc:268
#3  0x000000000467fea3 in dd::Entity_object_table_impl::restore_object_from_record (this=0x7f1b49f19470, otx=0x7f1b4c0a9970, record=..., o=0x7f1b4c0a9960)
    at ../../mysql-8.0.25/sql/dd/impl/types/entity_object_table_impl.cc:65
#4  0x00000000045e62f5 in dd::cache::Storage_adapter::get<dd::Item_name_key, dd::Abstract_table> (thd=0x7f1a94000f00, key=..., isolation=ISO_READ_COMMITTED,
    bypass_core_registry=false, object=0x7f1b4c0a9a80) at ../../mysql-8.0.25/sql/dd/impl/cache/storage_adapter.cc:189
#5  0x00000000045e0ff6 in dd::cache::Shared_dictionary_cache::get_uncached<dd::Item_name_key, dd::Abstract_table> (
    this=0x7daffe0 <dd::cache::Shared_dictionary_cache::instance()::s_cache>, thd=0x7f1a94000f00, key=..., isolation=ISO_READ_COMMITTED, object=0x7f1b4c0a9a80)
    at ../../mysql-8.0.25/sql/dd/impl/cache/shared_dictionary_cache.cc:113
#6  0x00000000045e0dde in dd::cache::Shared_dictionary_cache::get<dd::Item_name_key, dd::Abstract_table> (
    this=0x7daffe0 <dd::cache::Shared_dictionary_cache::instance()::s_cache>, thd=0x7f1a94000f00, key=..., element=0x7f1b4c0a9af8)
    at ../../mysql-8.0.25/sql/dd/impl/cache/shared_dictionary_cache.cc:98
#7  0x00000000045140c9 in dd::cache::Dictionary_client::acquire<dd::Item_name_key, dd::Abstract_table> (this=0x7f1a94004780, key=..., object=0x7f1b4c0a9b68,
    local_committed=0x7f1b4c0a9b67, local_uncommitted=0x7f1b4c0a9b66) at ../../mysql-8.0.25/sql/dd/impl/cache/dictionary_client.cc:883
#8  0x00000000044f5072 in dd::cache::Dictionary_client::acquire<dd::Abstract_table> (this=0x7f1a94004780, schema_name="test", object_name="b23", object=0x7f1b4c0a9ce8)
    at ../../mysql-8.0.25/sql/dd/impl/cache/dictionary_client.cc:1321
#9  0x00000000030a6786 in get_table_share (thd=0x7f1a94000f00, db=0x7f1a94c4f6b8 "test", table_name=0x7f1a94c4eae0 "b23", key=0x7f1b4c0aa5e7 "test", key_length=9,
    open_view=true, open_secondary=false) at ../../mysql-8.0.25/sql/sql_base.cc:765
#10 0x00000000030a6e7e in get_table_share_with_discover (thd=0x7f1a94000f00, table_list=0x7f1b4c0aa220, key=0x7f1b4c0aa5e7 "test", key_length=9, open_secondary=false,
    error=0x7f1b4c0aa0cc) at ../../mysql-8.0.25/sql/sql_base.cc:881
#11 0x00000000030ac317 in open_table (thd=0x7f1a94000f00, table_list=0x7f1b4c0aa220, ot_ctx=0x7f1b4c0aa1e0) at ../../mysql-8.0.25/sql/sql_base.cc:3199
#12 0x000000000327f976 in mysql_inplace_alter_table (thd=0x7f1a94000f00, schema=..., new_schema=..., table_def=0x0, altered_table_def=0x7f1a94ab3050, table_list=0x7f1a94c4f0a0,
    table=0x0, altered_table=0x7f1a94142ba0, ha_alter_info=0x7f1b4c0aaa70, inplace_supported=HA_ALTER_INPLACE_INSTANT, alter_ctx=0x7f1b4c0ab960,
    columns=std::set with 0 elements, fk_key_info=0x7f1a94c44ce8, fk_key_count=0, fk_invalidator=0x7f1b4c0ab890) at ../../mysql-8.0.25/sql/sql_table.cc:13291
#13 0x000000000328a3a6 in mysql_alter_table (thd=0x7f1a94000f00, new_db=0x7f1a94c4f6b8 "test", new_name=0x0, create_info=0x7f1b4c0ad440, table_list=0x7f1a94c4f0a0,
    alter_info=0x7f1b4c0ad2d0) at ../../mysql-8.0.25/sql/sql_table.cc:16924
#14 0x00000000037e6e3c in Sql_cmd_alter_table::execute (this=0x7f1a94c4f7d8, thd=0x7f1a94000f00) at ../../mysql-8.0.25/sql/sql_alter.cc:349
#15 0x00000000031b2496 in mysql_execute_command (thd=0x7f1a94000f00, first_level=true) at ../../mysql-8.0.25/sql/sql_parse.cc:4412
#16 0x00000000031b43b0 in dispatch_sql_command (thd=0x7f1a94000f00, parser_state=0x7f1b4c0aeb70) at ../../mysql-8.0.25/sql/sql_parse.cc:5000
#17 0x00000000031aa9d2 in dispatch_command (thd=0x7f1a94000f00, com_data=0x7f1b4c0afc20, command=COM_QUERY) at ../../mysql-8.0.25/sql/sql_parse.cc:1841
#18 0x00000000031a8e0c in do_command (thd=0x7f1a94000f00) at ../../mysql-8.0.25/sql/sql_parse.cc:1320
#19 0x000000000337b4a7 in handle_connection (arg=0x9a16410) at ../../mysql-8.0.25/sql/conn_handler/connection_handler_per_thread.cc:301
#20 0x0000000004f499d3 in pfs_spawn_thread (arg=0xb820270) at ../../../mysql-8.0.25/storage/perfschema/pfs.cc:2898
#21 0x00007f1b5eda86db in start_thread (arg=0x7f1b4c0b0700) at pthread_create.c:463
#22 0x00007f1b5cd6e71f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

``` 

mysql.columns表定义在 sql/dd/impl/tables/[columns.cc](<http://columns.cc>) 文件中

# 结论

查看执行计划: 

![image2021-8-2 12:15:31.png](/assets/01KJBYE2S5BW09A60PY13ARNZM/image2021-8-2%2012%3A15%3A31.png)

lower_case或其他因素影响了 col表的索引选择, 导致 col JOIN xxx 的输出顺序不同
