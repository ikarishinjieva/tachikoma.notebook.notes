---
title: 20210628 - Update 是如何进行的
confluence_page_id: 1147048
created_at: 2021-06-28T05:18:30+00:00
updated_at: 2021-06-28T05:18:30+00:00
---

# 入口堆栈
    
    
    #0  Sql_cmd_update::execute (this=0x7efb98006a00, thd=0x7efb98000dd0)
        at /opt/mysql-5.7.27/sql/sql_update.cc:3022
    #1  0x000055786bc5b8e8 in mysql_execute_command (thd=0x7efb98000dd0, first_level=true)
        at /opt/mysql-5.7.27/sql/sql_parse.cc:3606
    #2  0x000055786bc61640 in mysql_parse (thd=0x7efb98000dd0, parser_state=0x7efbfb34d530)
        at /opt/mysql-5.7.27/sql/sql_parse.cc:5570
    #3  0x000055786bc565a5 in dispatch_command (thd=0x7efb98000dd0, com_data=0x7efbfb34dde0,
        command=COM_QUERY) at /opt/mysql-5.7.27/sql/sql_parse.cc:1484
    #4  0x000055786bc5542f in do_command (thd=0x7efb98000dd0) at /opt/mysql-5.7.27/sql/sql_parse.cc:1025
    #5  0x000055786bd97f5e in handle_connection (arg=0x55786f6c3db0)
        at /opt/mysql-5.7.27/sql/conn_handler/connection_handler_per_thread.cc:306
    #6  0x000055786c492474 in pfs_spawn_thread (arg=0x557870251dc0)
        at /opt/mysql-5.7.27/storage/perfschema/pfs.cc:2190
    #7  0x00007efc1bfa46db in start_thread (arg=0x7efbfb34e700) at pthread_create.c:463
    #8  0x00007efc1b38e88f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

# 代码路径整理

  - Sql_cmd_update::execute

    - 对于multi update, execute_multi_table_update

    - 对于单表 update, try_single_table_update

      - update_precheck

      - open_tables_for_query

      - mysql_update_prepare_table

      - run_before_dml_hook

      - 对于ignore或strict mode, push_internal_handler

      - mysql_update

        - mysql_prepare_update

        - select_lex->get_optimizable_conditions

        - update.add_function_default_columns

        - substitute_gc

        - table->triggers->mark_fields

        - 处理分区表?

        - lock_tables

        - 对判断条件的处理

          - 处理CHECK OPTION

          - optimize_cond

          - explain_single_table_modification

          - substitute_for_best_equal_field

        - 处理分区表2 ?

        - table->init_cost_model

        - test_quick_select

        - table->update_const_key_parts

        - simple_remove_const

        - get_index_for_order

        - table->mark_columns_per_binlog_row_image

          - 根据ROW binlog的配置, 设置read_set/write_set (列是否出现在binlog中)

        - used_key_is_modified || order?

          - filesort ?

          - not filesort

            - table->file->try_semi_consistent_read(与innodb_locks_unsafe_for_binlog相关, 参数已弃用, 不再分析)

            - 初始化从引擎读取记录的方法: init_read_record / init_read_record_idx   
(init_read_record从七种方法中选取合适的 用于读取引擎记录 的方法,  
init_read_record_idx 用于Full index scan, 方法为: rr_index_first/rr_index_last, 其中会将读取方法更换成 rr_index )

            - stage开始: stage_searching_rows_for_update

            - while read_record {my_b_write(tempfile, ...)} : 从引擎中读取行主键(ref), 写入tempfile

            - … (不继续分析分支代码)

        - init_read_record

        - while

          - info.read_record

          - if !qep_tab.skip_record

            - store_record

            - fill_record_n_invoke_before_triggers

            - if (!records_are_comparable(table) || compare_records(table))

            - view check

            - table->file->ha_bulk_update_row / table->file->ha_update_row

            - table->triggers->process_triggers

        - 清理

      - 对于ignore或strict mdoe, pop_internal_handler

# read_record

以rr_quick为例 (SQL为 update a set b = 3 where a = 1 ), 调用堆栈为:
    
    
    #0  row_search_mvcc (buf=0x7f922000f8b0 "\377", mode=PAGE_CUR_GE, prebuilt=0x7f922001e200,
        match_mode=1, direction=0) at /opt/mysql-5.7.27/storage/innobase/row/row0sel.cc:4611
    #1  0x000055572737a621 in ha_innobase::index_read (this=0x7f922000f5c0, buf=0x7f922000f8b0 "\377",
        key_ptr=0x7f9220a43500 "\001", key_len=4, find_flag=HA_READ_KEY_EXACT)
        at /opt/mysql-5.7.27/storage/innobase/handler/ha_innodb.cc:8744
    #2  0x0000555726958ffc in handler::index_read_map (this=0x7f922000f5c0, buf=0x7f922000f8b0 "\377",
        key=0x7f9220a43500 "\001", keypart_map=1, find_flag=HA_READ_KEY_EXACT)
        at /opt/mysql-5.7.27/sql/handler.h:2809
    #3  0x0000555726949ad4 in handler::ha_index_read_map (this=0x7f922000f5c0,
        buf=0x7f922000f8b0 "\377", key=0x7f9220a43500 "\001", keypart_map=1,
        find_flag=HA_READ_KEY_EXACT) at /opt/mysql-5.7.27/sql/handler.cc:3039
    #4  0x00005557269545a0 in handler::read_range_first (this=0x7f922000f5c0, start_key=0x7f922000f6a8,
        end_key=0x7f922000f6c8, eq_range_arg=true, sorted=true) at /opt/mysql-5.7.27/sql/handler.cc:7404
    #5  0x00005557269523a6 in handler::multi_range_read_next (this=0x7f922000f5c0,
        range_info=0x7f92982ecb08) at /opt/mysql-5.7.27/sql/handler.cc:6469
    #6  0x00005557269532f7 in DsMrr_impl::dsmrr_next (this=0x7f922000f820, range_info=0x7f92982ecb08)
        at /opt/mysql-5.7.27/sql/handler.cc:6861
    #7  0x000055572738d82c in ha_innobase::multi_range_read_next (this=0x7f922000f5c0,
        range_info=0x7f92982ecb08) at /opt/mysql-5.7.27/storage/innobase/handler/ha_innodb.cc:20571
    #8  0x00005557271b4a5b in QUICK_RANGE_SELECT::get_next (this=0x7f92200223f0)
        at /opt/mysql-5.7.27/sql/opt_range.cc:11233
    #9  0x0000555726ece2b3 in rr_quick (info=0x7f92982ecda0) at /opt/mysql-5.7.27/sql/records.cc:398
    #10 0x0000555727077c64 in mysql_update (thd=0x7f9220000dd0, fields=..., values=...,
        limit=18446744073709551615, handle_duplicates=DUP_ERROR, found_return=0x7f92982ed348,
        updated_return=0x7f92982ed350) at /opt/mysql-5.7.27/sql/sql_update.cc:812
    #11 0x000055572707e6da in Sql_cmd_update::try_single_table_update (this=0x7f92200069e8,
        thd=0x7f9220000dd0, switch_to_multitable=0x7f92982ed3e7)
        at /opt/mysql-5.7.27/sql/sql_update.cc:2903
    #12 0x000055572707ec06 in Sql_cmd_update::execute (this=0x7f92200069e8, thd=0x7f9220000dd0)
        at /opt/mysql-5.7.27/sql/sql_update.cc:3030
    #13 0x0000555726fb78e8 in mysql_execute_command (thd=0x7f9220000dd0, first_level=true)
        at /opt/mysql-5.7.27/sql/sql_parse.cc:3606
    #14 0x0000555726fbd640 in mysql_parse (thd=0x7f9220000dd0, parser_state=0x7f92982ee530)
        at /opt/mysql-5.7.27/sql/sql_parse.cc:5570
    #15 0x0000555726fb25a5 in dispatch_command (thd=0x7f9220000dd0, com_data=0x7f92982eede0,
        command=COM_QUERY) at /opt/mysql-5.7.27/sql/sql_parse.cc:1484
    #16 0x0000555726fb142f in do_command (thd=0x7f9220000dd0) at /opt/mysql-5.7.27/sql/sql_parse.cc:1025
    #17 0x00005557270f3f5e in handle_connection (arg=0x55572a758210)
        at /opt/mysql-5.7.27/sql/conn_handler/connection_handler_per_thread.cc:306
    #18 0x00005557277ee474 in pfs_spawn_thread (arg=0x55572a7eb820)
        at /opt/mysql-5.7.27/storage/perfschema/pfs.cc:2190
    #19 0x00007f92a46d46db in start_thread (arg=0x7f92982ef700) at pthread_create.c:463
    #20 0x00007f92a3abe88f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95

从引擎获取的数据, 暂存在 table->record[0] :
    
    
    handler::read_range_first
    {...
        result= ha_index_read_map(table->record[0],
                                  start_key->key,
                                  start_key->keypart_map,
                                  start_key->flag);
    ...}

如何确定从哪里下一行?

  - 如何定位扫描到了哪一行, 使用的是 m_prebuilt->search_tuple

  - m_prebuilt 是 ha_innobase (handler的实现)的属性

## mysql_update

核心逻辑:
    
    
    while (true)
    {
      info.read_record(&info); //从引擎读取行, 到table->record[0]
      store_record(table,record[1]); //将table->record[0]存储到table->record[1]
      fill_record_n_invoke_before_triggers(... table, fields, values, ...);  //将fields-values中的更新, 更新到table->record[1]
        fill_record(... table, fields, values, ...);
          for field, value in fields, values: value->save_in_field(field)
      if record is changed (compare_records)
      {
        batch update: table->file->ha_bulk_update_row
        或
        non-batch update: table->file->ha_update_row
      }
    }

## ha_update_row

update_row

入口堆栈:
    
    
    #0  ha_innobase::update_row (this=0x7facf400f5c0, old_row=0x7facf400f8c0 "\375\001", new_row=0x7facf400f8b0 "\375\001")
        at /opt/mysql-5.7.27/storage/innobase/handler/ha_innodb.cc:8174
    #1  0x000055a2c111b506 in handler::ha_update_row (this=0x7facf400f5c0, old_data=0x7facf400f8c0 "\375\001", new_data=0x7facf400f8b0 "\375\001")
        at /opt/mysql-5.7.27/sql/handler.cc:8127
    #2  0x000055a2c183cedf in mysql_update (thd=0x7facf4000dd0, fields=..., values=..., limit=18446744073709551615, handle_duplicates=DUP_ERROR, found_return=0x7fad782f6348,
        updated_return=0x7fad782f6350) at /opt/mysql-5.7.27/sql/sql_update.cc:887
    #3  0x000055a2c18436da in Sql_cmd_update::try_single_table_update (this=0x7facf40069e8, thd=0x7facf4000dd0, switch_to_multitable=0x7fad782f63e7)
        at /opt/mysql-5.7.27/sql/sql_update.cc:2903
    #4  0x000055a2c1843c06 in Sql_cmd_update::execute (this=0x7facf40069e8, thd=0x7facf4000dd0) at /opt/mysql-5.7.27/sql/sql_update.cc:3030
    #5  0x000055a2c177c8e8 in mysql_execute_command (thd=0x7facf4000dd0, first_level=true) at /opt/mysql-5.7.27/sql/sql_parse.cc:3606
    #6  0x000055a2c1782640 in mysql_parse (thd=0x7facf4000dd0, parser_state=0x7fad782f7530) at /opt/mysql-5.7.27/sql/sql_parse.cc:5570
    #7  0x000055a2c17775a5 in dispatch_command (thd=0x7facf4000dd0, com_data=0x7fad782f7de0, command=COM_QUERY) at /opt/mysql-5.7.27/sql/sql_parse.cc:1484
    #8  0x000055a2c177642f in do_command (thd=0x7facf4000dd0) at /opt/mysql-5.7.27/sql/sql_parse.cc:1025
    #9  0x000055a2c18b8f5e in handle_connection (arg=0x55a2c4f467b0) at /opt/mysql-5.7.27/sql/conn_handler/connection_handler_per_thread.cc:306
    #10 0x000055a2c1fb3474 in pfs_spawn_thread (arg=0x55a2c4f532f0) at /opt/mysql-5.7.27/storage/perfschema/pfs.cc:2190
    #11 0x00007fad82ecf6db in start_thread (arg=0x7fad782f8700) at pthread_create.c:463
    #12 0x00007fad822b988f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
    
    
    update_row
    {
      calc_row_difference(..., old_row, new_row, m_upd_buf, ...); //将两行的差异列, 写入m_upd_buf
      
      innobase_srv_conc_enter_innodb(m_prebuilt); //进入innodb关键区
      row_update_for_mysql((byte*) old_row, m_prebuilt); //?
        row_update_for_mysql_using_upd_graph(..., prebuilt); //(old_row参数完全没用, 忽略)
          {
            thr = que_fork_get_first_thr(prebuilt->upd_graph);
            que_thr_move_to_run_state_for_mysql(thr, trx);
            row_upd_step(thr);
              row_upd
              {
                row_upd_clust_step //TODO
                  mtr_start
                  if (node->cmpl_info & UPD_NODE_NO_ORD_CHANGE) { //No secondary index change nor ordering fields change
                    row_upd_clust_rec
                  }
                  if (row_upd_changes_ord_field_binary) {
                    row_upd_clust_rec_by_insert
                  } else {
                    row_upd_clust_rec
                  }
                for each ?? index: row_upd_sec_step //TODO
              }
            que_thr_stop_for_mysql_no_error(thr, trx);
          }
      innobase_srv_conc_exit_innodb(m_prebuilt); //退出innodb关键区
    }

binlog_log_row

## TODO

  - select node performed the updates directly in-place
