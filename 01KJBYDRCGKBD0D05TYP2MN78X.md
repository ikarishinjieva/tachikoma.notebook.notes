---
title: 20210715 - 一个insert的p_s输出
confluence_page_id: 1343490
created_at: 2021-07-15T09:25:56+00:00
updated_at: 2021-07-15T13:18:16+00:00
---

# 单机insert

statement

```
master [localhost:19327] {msandbox} (performance_schema) > select * from events_statements_history_long\G
*************************** 1. row ***************************
              THREAD_ID: 30
               EVENT_ID: 17731
           END_EVENT_ID: 17842
             EVENT_NAME: statement/sql/insert
                 SOURCE:
            TIMER_START: 1064852719660000
              TIMER_END: 1064856665314000
             TIMER_WAIT: 3945654000
              LOCK_TIME: 141000000
               SQL_TEXT: insert into a values(2)
                 DIGEST: 8f90ef9afc29b87a1ea80bb9921fb423
            DIGEST_TEXT: INSERT INTO `a` VALUES (?)
         CURRENT_SCHEMA: test
            OBJECT_TYPE: NULL
          OBJECT_SCHEMA: NULL
            OBJECT_NAME: NULL
  OBJECT_INSTANCE_BEGIN: NULL
            MYSQL_ERRNO: 0
      RETURNED_SQLSTATE: 00000
           MESSAGE_TEXT: NULL
                 ERRORS: 0
               WARNINGS: 0
          ROWS_AFFECTED: 1
              ROWS_SENT: 0
          ROWS_EXAMINED: 0
CREATED_TMP_DISK_TABLES: 0
     CREATED_TMP_TABLES: 0
       SELECT_FULL_JOIN: 0
 SELECT_FULL_RANGE_JOIN: 0
           SELECT_RANGE: 0
     SELECT_RANGE_CHECK: 0
            SELECT_SCAN: 0
      SORT_MERGE_PASSES: 0
             SORT_RANGE: 0
              SORT_ROWS: 0
              SORT_SCAN: 0
          NO_INDEX_USED: 0
     NO_GOOD_INDEX_USED: 0
       NESTING_EVENT_ID: NULL
     NESTING_EVENT_TYPE: NULL
    NESTING_EVENT_LEVEL: 0
1 row in set (0.01 sec)
``` 

stage: 

```
select event_id, event_name, nesting_event_id from events_stages_history_long where nesting_event_id = 17731;
+----------+--------------------------------+------------------+
| event_id | event_name                     | nesting_event_id |
+----------+--------------------------------+------------------+
|    17732 | stage/sql/starting             |            17731 |
|    17738 | stage/sql/checking permissions |            17731 |
|    17740 | stage/sql/Opening tables       |            17731 |
|    17747 | stage/sql/init                 |            17731 |
|    17752 | stage/sql/System lock          |            17731 |
|    17758 | stage/sql/update               |            17731 |
|    17782 | stage/sql/end                  |            17731 |
|    17783 | stage/sql/query end            |            17731 |
|    17833 | stage/sql/closing tables       |            17731 |
|    17839 | stage/sql/freeing items        |            17731 |
|    17841 | stage/sql/cleaning up          |            17731 |
+----------+--------------------------------+------------------+
11 rows in set (0.00 sec)
``` 

wait: 

```
 select event_id, event_name, object_name, index_name, object_type, nesting_event_id, nesting_event_type, operation from events_waits_history_long where thread_id = 30;
+----------+---------------------------------------------------------+--------------------------------------------------------------+------------+-------------+------------------+--------------------+----------------+
| event_id | event_name                                              | object_name                                                  | index_name | object_type | nesting_event_id | nesting_event_type | operation      |
+----------+---------------------------------------------------------+--------------------------------------------------------------+------------+-------------+------------------+--------------------+----------------+
|    17730 | idle                                                    | NULL                                                         | NULL       | NULL        |             NULL | NULL               | idle           |
|    17733 | wait/io/socket/sql/client_connection                    | :0                                                           | NULL       | SOCKET      |            17732 | STAGE              | recv           |
|    17734 | wait/synch/mutex/sql/THD::LOCK_thd_data                 | NULL                                                         | NULL       | NULL        |            17732 | STAGE              | lock           |
|    17735 | wait/synch/mutex/sql/THD::LOCK_thd_query                | NULL                                                         | NULL       | NULL        |            17732 | STAGE              | lock           |
|    17736 | wait/synch/mutex/sql/THD::LOCK_thd_query                | NULL                                                         | NULL       | NULL        |            17732 | STAGE              | lock           |
|    17737 | wait/synch/mutex/sql/THD::LOCK_query_plan               | NULL                                                         | NULL       | NULL        |            17732 | STAGE              | lock           |
|    17739 | wait/synch/rwlock/sql/LOCK_grant                        | NULL                                                         | NULL       | NULL        |            17738 | STAGE              | read_lock      |
|    17741 | wait/synch/mutex/sql/LOCK_table_cache                   | NULL                                                         | NULL       | NULL        |            17740 | STAGE              | lock           |
|    17742 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17740 | STAGE              | lock           |
|    17743 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17740 | STAGE              | lock           |
|    17744 | wait/synch/mutex/sql/THD::LOCK_thd_data                 | NULL                                                         | NULL       | NULL        |            17740 | STAGE              | lock           |
|    17745 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17740 | STAGE              | lock           |
|    17746 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17740 | STAGE              | lock           |
|    17748 | wait/synch/rwlock/sql/LOCK_grant                        | NULL                                                         | NULL       | NULL        |            17747 | STAGE              | read_lock      |
|    17749 | wait/synch/mutex/sql/THD::LOCK_query_plan               | NULL                                                         | NULL       | NULL        |            17747 | STAGE              | lock           |
|    17750 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17747 | STAGE              | lock           |
|    17751 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17747 | STAGE              | lock           |
|    17754 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17753 | WAIT               | lock           |
|    17755 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17753 | WAIT               | lock           |
|    17757 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17753 | WAIT               | lock           |
|    17753 | wait/lock/table/sql/handler                             | a                                                            | PRIMARY    | TABLE       |            17752 | STAGE              | write external |
|    17760 | wait/synch/mutex/innodb/redo_rseg_mutex                 | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17761 | wait/synch/mutex/innodb/trx_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17762 | wait/synch/mutex/innodb/lock_mutex                      | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17763 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17764 | wait/synch/mutex/innodb/fil_system_mutex                | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17765 | wait/synch/sxlock/innodb/index_tree_rw_lock             | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | shared_lock    |
|    17766 | wait/synch/sxlock/innodb/hash_table_locks               | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | shared_lock    |
|    17767 | wait/synch/sxlock/innodb/hash_table_locks               | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | shared_lock    |
|    17768 | wait/synch/mutex/innodb/lock_mutex                      | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17769 | wait/synch/mutex/innodb/trx_undo_mutex                  | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17770 | wait/synch/mutex/innodb/redo_rseg_mutex                 | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17771 | wait/synch/sxlock/innodb/hash_table_locks               | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | shared_lock    |
|    17772 | wait/synch/mutex/innodb/log_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17773 | wait/synch/mutex/innodb/log_flush_order_mutex           | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17774 | wait/synch/mutex/innodb/flush_list_mutex                | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17775 | wait/synch/sxlock/innodb/hash_table_locks               | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | shared_lock    |
|    17776 | wait/synch/mutex/innodb/log_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17777 | wait/synch/mutex/innodb/log_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17778 | wait/synch/mutex/innodb/log_flush_order_mutex           | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17779 | wait/synch/mutex/innodb/flush_list_mutex                | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17780 | wait/synch/mutex/innodb/recalc_pool_mutex               | NULL                                                         | NULL       | NULL        |            17759 | WAIT               | lock           |
|    17759 | wait/io/table/sql/handler                               | a                                                            | NULL       | TABLE       |            17758 | STAGE              | insert         |
|    17781 | wait/synch/mutex/sql/THD::LOCK_query_plan               | NULL                                                         | NULL       | NULL        |            17758 | STAGE              | lock           |
|    17784 | wait/synch/mutex/sql/THD::LOCK_query_plan               | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17785 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17786 | wait/synch/mutex/innodb/redo_rseg_mutex                 | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17787 | wait/synch/sxlock/innodb/hash_table_locks               | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | shared_lock    |
|    17788 | wait/synch/mutex/innodb/log_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17789 | wait/synch/mutex/innodb/trx_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17790 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_flush_queue    | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17791 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_log            | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17792 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_flush_queue    | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17793 | wait/synch/mutex/sql/LOCK_plugin                        | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17794 | wait/synch/mutex/innodb/log_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17795 | wait/synch/mutex/innodb/log_sys_write_mutex             | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17796 | wait/synch/mutex/innodb/log_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17797 | wait/synch/mutex/innodb/fil_system_mutex                | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17798 | wait/io/file/innodb/innodb_log_file                     | /root/sandboxes/rsandbox_5_7_26/master/data/ib_logfile0      | NULL       | FILE        |            17783 | STAGE              | write          |
|    17799 | wait/synch/mutex/innodb/fil_system_mutex                | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17800 | wait/synch/mutex/innodb/fil_system_mutex                | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17801 | wait/io/file/innodb/innodb_log_file                     | /root/sandboxes/rsandbox_5_7_26/master/data/ib_logfile0      | NULL       | FILE        |            17783 | STAGE              | sync           |
|    17802 | wait/synch/mutex/innodb/fil_system_mutex                | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17803 | wait/synch/rwlock/sql/gtid_commit_rollback              | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | read_lock      |
|    17804 | wait/synch/mutex/sql/Gtid_state                         | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17805 | wait/io/file/sql/binlog                                 | /root/sandboxes/rsandbox_5_7_26/master/data/mysql-bin.000001 | NULL       | FILE        |            17783 | STAGE              | write          |
|    17806 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_sync_queue     | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17807 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_sync           | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17808 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_sync_queue     | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17809 | wait/io/file/sql/binlog                                 | /root/sandboxes/rsandbox_5_7_26/master/data/mysql-bin.000001 | NULL       | FILE        |            17783 | STAGE              | sync           |
|    17810 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_binlog_end_pos | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17811 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit_queue   | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17812 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit         | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17813 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit_queue   | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17814 | wait/synch/mutex/sql/LOCK_slave_trans_dep_tracker       | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17815 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17816 | wait/synch/mutex/innodb/redo_rseg_mutex                 | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17817 | wait/synch/sxlock/innodb/hash_table_locks               | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | shared_lock    |
|    17818 | wait/synch/sxlock/innodb/hash_table_locks               | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | shared_lock    |
|    17819 | wait/synch/mutex/innodb/log_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17820 | wait/synch/mutex/innodb/log_flush_order_mutex           | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17821 | wait/synch/mutex/innodb/flush_list_mutex                | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17822 | wait/synch/mutex/innodb/trx_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17823 | wait/synch/mutex/innodb/trx_sys_mutex                   | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17824 | wait/synch/mutex/innodb/lock_mutex                      | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17825 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17826 | wait/synch/mutex/innodb/redo_rseg_mutex                 | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17827 | wait/synch/mutex/innodb/redo_rseg_mutex                 | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17828 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17829 | wait/synch/rwlock/sql/gtid_commit_rollback              | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | read_lock      |
|    17830 | wait/synch/mutex/sql/Gtid_state                         | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17831 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_xids           | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17832 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_done           | NULL                                                         | NULL       | NULL        |            17783 | STAGE              | lock           |
|    17834 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17833 | STAGE              | lock           |
|    17835 | wait/synch/mutex/sql/THD::LOCK_thd_data                 | NULL                                                         | NULL       | NULL        |            17833 | STAGE              | lock           |
|    17836 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17833 | STAGE              | lock           |
|    17837 | wait/synch/mutex/innodb/trx_mutex                       | NULL                                                         | NULL       | NULL        |            17833 | STAGE              | lock           |
|    17838 | wait/synch/mutex/sql/LOCK_table_cache                   | NULL                                                         | NULL       | NULL        |            17833 | STAGE              | lock           |
|    17840 | wait/io/socket/sql/client_connection                    | :0                                                           | NULL       | SOCKET      |            17839 | STAGE              | send           |
|    17842 | wait/synch/mutex/sql/THD::LOCK_thd_query                | NULL                                                         | NULL       | NULL        |            17841 | STAGE              | lock           |
|    17843 | wait/synch/mutex/sql/THD::LOCK_thd_query                | NULL                                                         | NULL       | NULL        |            17731 | STATEMENT          | lock           |
+----------+---------------------------------------------------------+--------------------------------------------------------------+------------+-------------+------------------+--------------------+----------------+
``` 

整理: 

  - statement [17731]
    - stage/sql/starting [17732]

      - wait/io/socket/sql/client_connection (recv) [17733]
      - wait/synch/mutex/sql/THD::LOCK_thd_data [17734]

      - wait/synch/mutex/sql/THD::LOCK_thd_query [17735]

      - wait/synch/mutex/sql/THD::LOCK_thd_query [17736]

      - wait/synch/mutex/sql/THD::LOCK_query_plan [17737]

    - stage/sql/checking permissions [17738]

      - wait/synch/rwlock/sql/LOCK_grant (read_lock) [17739]
    - stage/sql/Opening tables [17740]

      - wait/synch/mutex/sql/LOCK_table_cache [17741]

      - wait/synch/mutex/innodb/trx_mutex [17742]

      - wait/synch/mutex/innodb/trx_mutex [17743]

      - wait/synch/mutex/sql/THD::LOCK_thd_data [17744]

      - wait/synch/mutex/innodb/trx_mutex [17745]

      - wait/synch/mutex/innodb/trx_mutex [17746]

    - stage/sql/init [17747]

      - wait/synch/rwlock/sql/LOCK_grant (read_lock) [17748]

      - wait/synch/mutex/sql/THD::LOCK_query_plan [17749]

      - wait/synch/mutex/innodb/trx_mutex [17750]

      - wait/synch/mutex/innodb/trx_mutex [17751]

    - stage/sql/System lock [17752]

      - wait/lock/table/sql/handler (table a, write external) [17753]

        - wait/synch/mutex/innodb/trx_mutex [17754]

        - wait/synch/mutex/innodb/trx_mutex [17755]

        - wait/synch/mutex/innodb/trx_mutex [17757]

    - stage/sql/update [17758]

      - wait/io/table/sql/handler (table a, insert) [17759]
        - wait/synch/mutex/innodb/redo_rseg_mutex [17760]

        - wait/synch/mutex/innodb/trx_sys_mutex [17761]

        - wait/synch/mutex/innodb/lock_mutex [17762]

        - wait/synch/mutex/innodb/trx_mutex [17763]

        - wait/synch/mutex/innodb/fil_system_mutex [17764]

        - wait/synch/sxlock/innodb/index_tree_rw_lock (shared_lock) [17765] 

        - wait/synch/sxlock/innodb/hash_table_locks (shared_lock) [17766]

        - wait/synch/sxlock/innodb/hash_table_locks (shared_lock) [17767]

        - wait/synch/mutex/innodb/lock_mutex [17768]

        - wait/synch/mutex/innodb/trx_undo_mutex [17769]

        - wait/synch/mutex/innodb/redo_rseg_mutex [17770]

        - wait/synch/sxlock/innodb/hash_table_locks (shared_lock) [17771]

        - wait/synch/mutex/innodb/log_sys_mutex [17772]

        - wait/synch/mutex/innodb/log_flush_order_mutex [17773]

        - wait/synch/mutex/innodb/flush_list_mutex [17774]

        - wait/synch/sxlock/innodb/hash_table_locks (shared_lock) [17775]

        - wait/synch/mutex/innodb/log_sys_mutex [17776]

        - wait/synch/mutex/innodb/log_sys_mutex [17777]

        - wait/synch/mutex/innodb/log_flush_order_mutex [17778]

        - wait/synch/mutex/innodb/flush_list_mutex [17779]

        - wait/synch/mutex/innodb/recalc_pool_mutex [17780]

      - wait/synch/mutex/sql/THD::LOCK_query_plan [17781]
        - wait/synch/mutex/sql/THD::LOCK_query_plan [17784]

        - wait/synch/mutex/innodb/trx_mutex [17785]

        - wait/synch/mutex/innodb/redo_rseg_mutex [17786]

        - wait/synch/sxlock/innodb/hash_table_locks (shared_lock) [17787]

        - wait/synch/mutex/innodb/log_sys_mutex [17788]

        - wait/synch/mutex/innodb/trx_sys_mutex [17789]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_flush_queue [17790]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_log [17791]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_flush_queue [17792]

        - wait/synch/mutex/sql/LOCK_plugin [17793]

        - wait/synch/mutex/innodb/log_sys_mutex [17794]

        - wait/synch/mutex/innodb/log_sys_write_mutex [17795]

        - wait/synch/mutex/innodb/log_sys_mutex [17796]

        - wait/synch/mutex/innodb/fil_system_mutex [17797]

        - wait/io/file/innodb/innodb_log_file (ib_logfile0, write) [17798]

        - wait/synch/mutex/innodb/fil_system_mutex [17799]

        - wait/synch/mutex/innodb/fil_system_mutex [17800]

        - wait/io/file/innodb/innodb_log_file (ib_logfile0, sync) [17801]

        - wait/synch/mutex/innodb/fil_system_mutex [17802]

        - wait/synch/rwlock/sql/gtid_commit_rollback [17803]

        - wait/synch/mutex/sql/Gtid_state | [17804]

        - wait/io/file/sql/binlog (mysql-bin.000001, write) [17805]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_sync_queue [17806]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_sync [17807]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_sync_queue [17808]

        - wait/io/file/sql/binlog (mysql-bin.000001, sync) [17809]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_binlog_end_pos [17810]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit_queue [17811]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit [17812]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit_queue [17813]

        - wait/synch/mutex/sql/LOCK_slave_trans_dep_tracker | [17814]

        - wait/synch/mutex/innodb/trx_mutex | [17815]

        - wait/synch/mutex/innodb/redo_rseg_mutex | [17816]

        - wait/synch/sxlock/innodb/hash_table_locks (shared_lock) [17817]

        - wait/synch/sxlock/innodb/hash_table_locks (shared_lock) [17818]

        - wait/synch/mutex/innodb/log_sys_mutex | [17819]

        - wait/synch/mutex/innodb/log_flush_order_mutex | [17820]

        - wait/synch/mutex/innodb/flush_list_mutex | [17821]

        - wait/synch/mutex/innodb/trx_sys_mutex | [17822]

        - wait/synch/mutex/innodb/trx_sys_mutex | [17823]

        - wait/synch/mutex/innodb/lock_mutex | [17824]

        - wait/synch/mutex/innodb/trx_mutex | [17825]

        - wait/synch/mutex/innodb/redo_rseg_mutex | [17826]

        - wait/synch/mutex/innodb/redo_rseg_mutex | [17827]

        - wait/synch/mutex/innodb/trx_mutex | [17828]

        - wait/synch/rwlock/sql/gtid_commit_rollback (read_lock) [17829]

        - wait/synch/mutex/sql/Gtid_state | [17830]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_xids [17831]

        - wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_done [17832]

    - stage/sql/end [17782]

    - stage/sql/query end [17783]

    - stage/sql/closing tables [17833]

      - wait/synch/mutex/innodb/trx_mutex [17834]

      - wait/synch/mutex/sql/THD::LOCK_thd_data [17835]

      - wait/synch/mutex/innodb/trx_mutex [17836]

      - wait/synch/mutex/innodb/trx_mutex [17837]

      - wait/synch/mutex/sql/LOCK_table_cache [17838]

    - stage/sql/freeing items [17839]

      - wait/io/socket/sql/client_connection (send) [17840]
    - stage/sql/cleaning up [17841]

      - wait/synch/mutex/sql/THD::LOCK_thd_query [17842]
    - wait/synch/mutex/sql/THD::LOCK_thd_query [17843]

# 半同步

stages: 

```
select * from events_stages_history_long;
+-----------+----------+--------------+-------------------------------------------------------------------------+--------+-----------------+-----------------+---------------+----------------+----------------+------------------+--------------------+
| THREAD_ID | EVENT_ID | END_EVENT_ID | EVENT_NAME                                                              | SOURCE | TIMER_START     | TIMER_END       | TIMER_WAIT    | WORK_COMPLETED | WORK_ESTIMATED | NESTING_EVENT_ID | NESTING_EVENT_TYPE |
+-----------+----------+--------------+-------------------------------------------------------------------------+--------+-----------------+-----------------+---------------+----------------+----------------+------------------+--------------------+
|        34 |     1300 |         1305 | stage/sql/starting                                                      |        | 780216221388000 | 780216403323000 |     181935000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1306 |         1307 | stage/sql/checking permissions                                          |        | 780216403323000 | 780216410908000 |       7585000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1308 |         1322 | stage/sql/Opening tables                                                |        | 780216410908000 | 780216465135000 |      54227000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1323 |         1327 | stage/sql/init                                                          |        | 780216465135000 | 780216478829000 |      13694000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1328 |         1333 | stage/sql/System lock                                                   |        | 780216478829000 | 780216485488000 |       6659000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1334 |         1357 | stage/sql/update                                                        |        | 780216485488000 | 780216600943000 |     115455000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1358 |         1358 | stage/sql/end                                                           |        | 780216600943000 | 780216602453000 |       1510000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1359 |         1399 | stage/sql/query end                                                     |        | 780216602453000 | 780221774227000 |    5171774000 |           NULL |           NULL |             1299 | STATEMENT          |
|        32 |      291 |          293 | stage/sql/Master has sent all binlog to slave; waiting for more updates |        | 770294501480000 | 780221808016000 | 9927306536000 |           NULL |           NULL |               12 | STATEMENT          |
|        32 |      294 |          350 | stage/sql/Sending binlog event to slave                                 |        | 780221808016000 | 780221973423000 |     165407000 |           NULL |           NULL |               12 | STATEMENT          |
|        34 |     1400 |         1402 | stage/semisync/Waiting for semi-sync ACK from slave                     |        | 780221774227000 | 780222254341000 |     480114000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1403 |         1426 | stage/sql/query end                                                     |        | 780222254341000 | 780222345899000 |      91558000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1427 |         1432 | stage/sql/closing tables                                                |        | 780222345899000 | 780222363407000 |      17508000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1433 |         1434 | stage/sql/freeing items                                                 |        | 780222363407000 | 780222420648000 |      57241000 |           NULL |           NULL |             1299 | STATEMENT          |
|        34 |     1435 |         1436 | stage/sql/cleaning up                                                   |        | 780222420648000 | 780222422760000 |       2112000 |           NULL |           NULL |             1299 | STATEMENT          |
+-----------+----------+--------------+-------------------------------------------------------------------------+--------+-----------------+-----------------+---------------+----------------+----------------+------------------+--------------------+

``` 

waits:

```
select thread_id, event_id, event_name, timer_start, object_name, operation  from events_waits_history_long ;
+-----------+----------+---------------------------------------------------------+-----------------+--------------------------------------------------------------+----------------+
| thread_id | event_id | event_name                                              | timer_start     | object_name                                                  | operation      |
+-----------+----------+---------------------------------------------------------+-----------------+--------------------------------------------------------------+----------------+
|        34 |     1298 | idle                                                    | 770295120000000 | NULL                                                         | idle           |
|        34 |     1301 | wait/io/socket/sql/client_connection                    | 780814135864620 | :0                                                           | recv           |
|        34 |     1302 | wait/synch/mutex/sql/THD::LOCK_thd_data                 | 780814247410428 | NULL                                                         | lock           |
|        34 |     1303 | wait/synch/mutex/sql/THD::LOCK_thd_query                | 780814249801388 | NULL                                                         | lock           |
|        34 |     1304 | wait/synch/mutex/sql/THD::LOCK_thd_query                | 780814262819580 | NULL                                                         | lock           |
|        34 |     1305 | wait/synch/mutex/sql/THD::LOCK_query_plan               | 780814311124496 | NULL                                                         | lock           |
|        34 |     1307 | wait/synch/rwlock/sql/LOCK_grant                        | 780814320258632 | NULL                                                         | read_lock      |
|        34 |     1309 | wait/synch/mutex/sql/LOCK_table_cache                   | 780814345755796 | NULL                                                         | lock           |
|        34 |     1310 | wait/synch/mutex/innodb/trx_mutex                       | 780814354058112 | NULL                                                         | lock           |
|        34 |     1311 | wait/synch/mutex/innodb/trx_mutex                       | 780814355057132 | NULL                                                         | lock           |
|        34 |     1312 | wait/synch/mutex/sql/THD::LOCK_thd_data                 | 780814355691656 | NULL                                                         | lock           |
|        34 |     1313 | wait/synch/mutex/innodb/trx_mutex                       | 780814358774824 | NULL                                                         | lock           |
|        34 |     1314 | wait/synch/mutex/innodb/trx_mutex                       | 780814359073276 | NULL                                                         | lock           |
|        34 |     1315 | wait/synch/mutex/innodb/trx_mutex                       | 780814361860500 | NULL                                                         | lock           |
|        34 |     1316 | wait/synch/mutex/innodb/trx_mutex                       | 780814362052780 | NULL                                                         | lock           |
|        34 |     1317 | wait/synch/mutex/innodb/trx_mutex                       | 780814362773412 | NULL                                                         | lock           |
|        34 |     1318 | wait/synch/mutex/innodb/dict_sys_mutex                  | 780814372294616 | NULL                                                         | lock           |
|        34 |     1319 | wait/synch/mutex/innodb/trx_mutex                       | 780814373164056 | NULL                                                         | lock           |
|        34 |     1320 | wait/synch/rwlock/sql/Trans_delegate::lock              | 780814375439648 | NULL                                                         | read_lock      |
|        34 |     1321 | wait/synch/mutex/sql/LOCK_plugin                        | 780814376030700 | NULL                                                         | lock           |
|        34 |     1322 | wait/synch/mutex/sql/LOCK_plugin                        | 780814377645016 | NULL                                                         | lock           |
|        34 |     1324 | wait/synch/rwlock/sql/LOCK_grant                        | 780814381653636 | NULL                                                         | read_lock      |
|        34 |     1325 | wait/synch/mutex/sql/THD::LOCK_query_plan               | 780814387906080 | NULL                                                         | lock           |
|        34 |     1326 | wait/synch/mutex/innodb/trx_mutex                       | 780814391522616 | NULL                                                         | lock           |
|        34 |     1327 | wait/synch/mutex/innodb/trx_mutex                       | 780814391809364 | NULL                                                         | lock           |
|        34 |     1330 | wait/synch/mutex/innodb/trx_mutex                       | 780814393255644 | NULL                                                         | lock           |
|        34 |     1331 | wait/synch/mutex/innodb/trx_mutex                       | 780814393534868 | NULL                                                         | lock           |
|        34 |     1333 | wait/synch/mutex/innodb/trx_mutex                       | 780814394976132 | NULL                                                         | lock           |
|        34 |     1329 | wait/lock/table/sql/handler                             | 780814392988960 | a                                                            | write external |
|        34 |     1336 | wait/synch/mutex/innodb/redo_rseg_mutex                 | 780814408124740 | NULL                                                         | lock           |
|        34 |     1337 | wait/synch/mutex/innodb/trx_sys_mutex                   | 780814408879648 | NULL                                                         | lock           |
|        34 |     1338 | wait/synch/mutex/innodb/lock_mutex                      | 780814415129584 | NULL                                                         | lock           |
|        34 |     1339 | wait/synch/mutex/innodb/trx_mutex                       | 780814415513308 | NULL                                                         | lock           |
|        34 |     1340 | wait/synch/mutex/innodb/fil_system_mutex                | 780814422031600 | NULL                                                         | lock           |
|        34 |     1341 | wait/synch/sxlock/innodb/index_tree_rw_lock             | 780814423996200 | NULL                                                         | shared_lock    |
|        34 |     1342 | wait/synch/sxlock/innodb/hash_table_locks               | 780814427106956 | NULL                                                         | shared_lock    |
|        34 |     1343 | wait/synch/sxlock/innodb/hash_table_locks               | 780814432329448 | NULL                                                         | shared_lock    |
|        34 |     1344 | wait/synch/mutex/innodb/lock_mutex                      | 780814438227428 | NULL                                                         | lock           |
|        34 |     1345 | wait/synch/mutex/innodb/trx_undo_mutex                  | 780814440763016 | NULL                                                         | lock           |
|        34 |     1346 | wait/synch/mutex/innodb/redo_rseg_mutex                 | 780814441669240 | NULL                                                         | lock           |
|        34 |     1347 | wait/synch/sxlock/innodb/hash_table_locks               | 780814442414116 | NULL                                                         | shared_lock    |
|        34 |     1348 | wait/synch/mutex/innodb/log_sys_mutex                   | 780814451822460 | NULL                                                         | lock           |
|        34 |     1349 | wait/synch/mutex/innodb/log_flush_order_mutex           | 780814453248676 | NULL                                                         | lock           |
|        34 |     1350 | wait/synch/mutex/innodb/flush_list_mutex                | 780814454414896 | NULL                                                         | lock           |
|        34 |     1351 | wait/synch/sxlock/innodb/hash_table_locks               | 780814455518416 | NULL                                                         | shared_lock    |
|        34 |     1352 | wait/synch/mutex/innodb/log_sys_mutex                   | 780814457130224 | NULL                                                         | lock           |
|        34 |     1353 | wait/synch/mutex/innodb/log_sys_mutex                   | 780814466728340 | NULL                                                         | lock           |
|        34 |     1354 | wait/synch/mutex/innodb/log_flush_order_mutex           | 780814467810124 | NULL                                                         | lock           |
|        34 |     1355 | wait/synch/mutex/innodb/flush_list_mutex                | 780814468022468 | NULL                                                         | lock           |
|        34 |     1356 | wait/synch/mutex/innodb/recalc_pool_mutex               | 780814469668552 | NULL                                                         | lock           |
|        34 |     1335 | wait/io/table/sql/handler                               | 780814403907956 | a                                                            | insert         |
|        34 |     1357 | wait/synch/mutex/sql/THD::LOCK_query_plan               | 780814510123428 | NULL                                                         | lock           |
|        34 |     1360 | wait/synch/mutex/sql/THD::LOCK_query_plan               | 780814516730336 | NULL                                                         | lock           |
|        34 |     1361 | wait/synch/mutex/innodb/trx_mutex                       | 780814520565068 | NULL                                                         | lock           |
|        34 |     1362 | wait/synch/mutex/innodb/redo_rseg_mutex                 | 780814521073356 | NULL                                                         | lock           |
|        34 |     1363 | wait/synch/sxlock/innodb/hash_table_locks               | 780814521676112 | NULL                                                         | shared_lock    |
|        34 |     1364 | wait/synch/mutex/innodb/log_sys_mutex                   | 780814522919244 | NULL                                                         | lock           |
|        34 |     1365 | wait/synch/mutex/innodb/trx_sys_mutex                   | 780814523996848 | NULL                                                         | lock           |
|        34 |     1366 | wait/synch/rwlock/sql/Trans_delegate::lock              | 780814527278984 | NULL                                                         | read_lock      |
|        34 |     1367 | wait/synch/mutex/sql/LOCK_plugin                        | 780814527579108 | NULL                                                         | lock           |
|        34 |     1368 | wait/synch/mutex/sql/LOCK_plugin                        | 780814528075692 | NULL                                                         | lock           |
|        34 |     1369 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_flush_queue    | 780814529133232 | NULL                                                         | lock           |
|        34 |     1370 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_log            | 780814529895664 | NULL                                                         | lock           |
|        34 |     1371 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_flush_queue    | 780814530258488 | NULL                                                         | lock           |
|        34 |     1372 | wait/synch/mutex/sql/LOCK_plugin                        | 780814530979120 | NULL                                                         | lock           |
|        34 |     1373 | wait/synch/mutex/innodb/log_sys_mutex                   | 780814533870844 | NULL                                                         | lock           |
|        34 |     1374 | wait/synch/mutex/innodb/log_sys_write_mutex             | 780814534405884 | NULL                                                         | lock           |
|        34 |     1375 | wait/synch/mutex/innodb/log_sys_mutex                   | 780814534719384 | NULL                                                         | lock           |
|        34 |     1376 | wait/synch/mutex/innodb/fil_system_mutex                | 780814541289508 | NULL                                                         | lock           |
|        34 |     1377 | wait/io/file/innodb/innodb_log_file                     | 780814543352756 | /root/sandboxes/rsandbox_5_7_26/master/data/ib_logfile0      | write          |
|        34 |     1378 | wait/synch/mutex/innodb/fil_system_mutex                | 780814591592464 | NULL                                                         | lock           |
|        34 |     1379 | wait/synch/mutex/innodb/fil_system_mutex                | 780814593078872 | NULL                                                         | lock           |
|        34 |     1380 | wait/io/file/innodb/innodb_log_file                     | 780814593612240 | /root/sandboxes/rsandbox_5_7_26/master/data/ib_logfile0      | sync           |
|        34 |     1381 | wait/synch/mutex/innodb/fil_system_mutex                | 780817472449300 | NULL                                                         | lock           |
|        34 |     1382 | wait/synch/rwlock/sql/gtid_commit_rollback              | 780817494416036 | NULL                                                         | read_lock      |
|        34 |     1383 | wait/synch/mutex/sql/Gtid_state                         | 780817497550200 | NULL                                                         | lock           |
|        34 |     1384 | wait/io/file/sql/binlog                                 | 780817525970020 | /root/sandboxes/rsandbox_5_7_26/master/data/mysql-bin.000001 | write          |
|        34 |     1385 | wait/synch/rwlock/sql/Binlog_storage_delegate::lock     | 780817576531300 | NULL                                                         | read_lock      |
|        34 |     1386 | wait/synch/mutex/sql/LOCK_plugin                        | 780817577748516 | NULL                                                         | lock           |
|        34 |     1387 | wait/synch/mutex/semisync/LOCK_binlog_                  | 780817586408640 | NULL                                                         | lock           |
|        34 |     1388 | wait/synch/mutex/sql/LOCK_plugin                        | 780817592501408 | NULL                                                         | lock           |
|        34 |     1389 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_sync_queue     | 780817593523000 | NULL                                                         | lock           |
|        34 |     1390 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_sync           | 780817595030308 | NULL                                                         | lock           |
|        34 |     1391 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_sync_queue     | 780817596041868 | NULL                                                         | lock           |
|        34 |     1392 | wait/io/file/sql/binlog                                 | 780817596579416 | /root/sandboxes/rsandbox_5_7_26/master/data/mysql-bin.000001 | sync           |
|        34 |     1393 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_binlog_end_pos | 780819647609276 | NULL                                                         | lock           |
|        34 |     1394 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit_queue   | 780819670934512 | NULL                                                         | lock           |
|        34 |     1395 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit         | 780819672792940 | NULL                                                         | lock           |
|        34 |     1396 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit_queue   | 780819673122324 | NULL                                                         | lock           |
|        34 |     1397 | wait/synch/rwlock/sql/Binlog_storage_delegate::lock     | 780819676468832 | NULL                                                         | read_lock      |
|        34 |     1398 | wait/synch/mutex/sql/LOCK_plugin                        | 780819679535280 | NULL                                                         | lock           |
|        34 |     1399 | wait/synch/mutex/semisync/LOCK_binlog_                  | 780819684522856 | NULL                                                         | lock           |
|        32 |      292 | wait/synch/cond/sql/MYSQL_BIN_LOG::update_cond          | 770884815929404 | NULL                                                         | timed_wait     |
|        32 |      293 | wait/synch/mutex/sql/THD::LOCK_current_cond             | 780819720749244 | NULL                                                         | lock           |
|        32 |      295 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_binlog_end_pos | 780819727002524 | NULL                                                         | lock           |
|        32 |      296 | wait/io/file/sql/binlog                                 | 780819731801164 | /root/sandboxes/rsandbox_5_7_26/master/data/mysql-bin.000001 | read           |
|        32 |      297 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819767046088 | NULL                                                         | read_lock      |
|        32 |      298 | wait/synch/mutex/sql/LOCK_plugin                        | 780819769296600 | NULL                                                         | lock           |
|        32 |      299 | wait/synch/mutex/sql/LOCK_plugin                        | 780819773861996 | NULL                                                         | lock           |
|        32 |      300 | wait/synch/rwlock/sql/gtid_commit_rollback              | 780819777480204 | NULL                                                         | read_lock      |
|        32 |      301 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819778383084 | NULL                                                         | read_lock      |
|        32 |      302 | wait/synch/mutex/sql/LOCK_plugin                        | 780819778671504 | NULL                                                         | lock           |
|        32 |      303 | wait/synch/mutex/semisync/LOCK_binlog_                  | 780819780654496 | NULL                                                         | lock           |
|        32 |      304 | wait/synch/mutex/sql/LOCK_plugin                        | 780819782360772 | NULL                                                         | lock           |
|        32 |      305 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819795529444 | NULL                                                         | read_lock      |
|        32 |      306 | wait/synch/mutex/sql/LOCK_plugin                        | 780819795773556 | NULL                                                         | lock           |
|        32 |      307 | wait/synch/mutex/sql/LOCK_plugin                        | 780819796800164 | NULL                                                         | lock           |
|        32 |      308 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819797221508 | NULL                                                         | read_lock      |
|        32 |      309 | wait/synch/mutex/sql/LOCK_plugin                        | 780819797424656 | NULL                                                         | lock           |
|        32 |      310 | wait/synch/mutex/sql/LOCK_plugin                        | 780819797744008 | NULL                                                         | lock           |
|        32 |      311 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819798248952 | NULL                                                         | read_lock      |
|        32 |      312 | wait/synch/mutex/sql/LOCK_plugin                        | 780819798424512 | NULL                                                         | lock           |
|        32 |      313 | wait/synch/mutex/semisync/LOCK_binlog_                  | 780819798771452 | NULL                                                         | lock           |
|        32 |      314 | wait/synch/mutex/sql/LOCK_plugin                        | 780819799196976 | NULL                                                         | lock           |
|        32 |      315 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819799600764 | NULL                                                         | read_lock      |
|        32 |      316 | wait/synch/mutex/sql/LOCK_plugin                        | 780819799771308 | NULL                                                         | lock           |
|        32 |      317 | wait/synch/mutex/sql/LOCK_plugin                        | 780819800138312 | NULL                                                         | lock           |
|        32 |      318 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819800628208 | NULL                                                         | read_lock      |
|        32 |      319 | wait/synch/mutex/sql/LOCK_plugin                        | 780819800787048 | NULL                                                         | lock           |
|        32 |      320 | wait/synch/mutex/sql/LOCK_plugin                        | 780819801109744 | NULL                                                         | lock           |
|        32 |      321 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819801653980 | NULL                                                         | read_lock      |
|        32 |      322 | wait/synch/mutex/sql/LOCK_plugin                        | 780819801823688 | NULL                                                         | lock           |
|        32 |      323 | wait/synch/mutex/semisync/LOCK_binlog_                  | 780819802109600 | NULL                                                         | lock           |
|        32 |      324 | wait/synch/mutex/sql/LOCK_plugin                        | 780819802477440 | NULL                                                         | lock           |
|        32 |      325 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819802812676 | NULL                                                         | read_lock      |
|        32 |      326 | wait/synch/mutex/sql/LOCK_plugin                        | 780819802981548 | NULL                                                         | lock           |
|        32 |      327 | wait/synch/mutex/sql/LOCK_plugin                        | 780819803298392 | NULL                                                         | lock           |
|        32 |      328 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819803524948 | NULL                                                         | read_lock      |
|        32 |      329 | wait/synch/mutex/sql/LOCK_plugin                        | 780819803684624 | NULL                                                         | lock           |
|        32 |      330 | wait/synch/mutex/sql/LOCK_plugin                        | 780819803918704 | NULL                                                         | lock           |
|        32 |      331 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819804318312 | NULL                                                         | read_lock      |
|        32 |      332 | wait/synch/mutex/sql/LOCK_plugin                        | 780819804472972 | NULL                                                         | lock           |
|        32 |      333 | wait/synch/mutex/semisync/LOCK_binlog_                  | 780819804734640 | NULL                                                         | lock           |
|        32 |      334 | wait/synch/mutex/sql/LOCK_plugin                        | 780819805103316 | NULL                                                         | lock           |
|        32 |      335 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819805481188 | NULL                                                         | read_lock      |
|        32 |      336 | wait/synch/mutex/sql/LOCK_plugin                        | 780819805635848 | NULL                                                         | lock           |
|        32 |      337 | wait/synch/mutex/sql/LOCK_plugin                        | 780819805959380 | NULL                                                         | lock           |
|        32 |      338 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819806167544 | NULL                                                         | read_lock      |
|        32 |      339 | wait/synch/mutex/sql/LOCK_plugin                        | 780819806325548 | NULL                                                         | lock           |
|        32 |      340 | wait/synch/mutex/sql/LOCK_plugin                        | 780819806569660 | NULL                                                         | lock           |
|        32 |      341 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819807267720 | NULL                                                         | read_lock      |
|        32 |      342 | wait/synch/mutex/sql/LOCK_plugin                        | 780819807428232 | NULL                                                         | lock           |
|        32 |      343 | wait/synch/mutex/semisync/LOCK_binlog_                  | 780819807848740 | NULL                                                         | lock           |
|        32 |      344 | wait/synch/mutex/sql/LOCK_plugin                        | 780819809771540 | NULL                                                         | lock           |
|        32 |      345 | wait/synch/rwlock/sql/Binlog_transmit_delegate::lock    | 780819810158608 | NULL                                                         | read_lock      |
|        32 |      346 | wait/synch/mutex/sql/LOCK_plugin                        | 780819810319120 | NULL                                                         | lock           |
|        32 |      347 | wait/io/socket/sql/client_connection                    | 780819812530340 | 127.0.0.1:53664                                              | send           |
|        32 |      348 | wait/synch/mutex/sql/LOCK_plugin                        | 780819888643124 | NULL                                                         | lock           |
|        32 |      349 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_binlog_end_pos | 780819889276812 | NULL                                                         | lock           |
|        32 |      350 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_binlog_end_pos | 780819889989920 | NULL                                                         | lock           |
|        34 |     1401 | wait/synch/cond/semisync/COND_binlog_send_              | 780819698866944 | NULL                                                         | timed_wait     |
|        34 |     1402 | wait/synch/mutex/sql/THD::LOCK_current_cond             | 780820169576744 | NULL                                                         | lock           |
|        34 |     1404 | wait/synch/mutex/sql/LOCK_plugin                        | 780820176186996 | NULL                                                         | lock           |
|        34 |     1405 | wait/synch/mutex/sql/LOCK_slave_trans_dep_tracker       | 780820177891600 | NULL                                                         | lock           |
|        34 |     1406 | wait/synch/mutex/innodb/trx_mutex                       | 780820188621660 | NULL                                                         | lock           |
|        34 |     1407 | wait/synch/mutex/innodb/redo_rseg_mutex                 | 780820198003252 | NULL                                                         | lock           |
|        34 |     1408 | wait/synch/sxlock/innodb/hash_table_locks               | 780820201505256 | NULL                                                         | shared_lock    |
|        34 |     1409 | wait/synch/sxlock/innodb/hash_table_locks               | 780820212290492 | NULL                                                         | shared_lock    |
|        34 |     1410 | wait/synch/mutex/innodb/log_sys_mutex                   | 780820216433708 | NULL                                                         | lock           |
|        34 |     1411 | wait/synch/mutex/innodb/log_flush_order_mutex           | 780820218538756 | NULL                                                         | lock           |
|        34 |     1412 | wait/synch/mutex/innodb/flush_list_mutex                | 780820220280980 | NULL                                                         | lock           |
|        34 |     1413 | wait/synch/mutex/innodb/trx_sys_mutex                   | 780820222361784 | NULL                                                         | lock           |
|        34 |     1414 | wait/synch/mutex/innodb/trx_sys_mutex                   | 780820226693100 | NULL                                                         | lock           |
|        34 |     1415 | wait/synch/mutex/innodb/lock_mutex                      | 780820227306724 | NULL                                                         | lock           |
|        34 |     1416 | wait/synch/mutex/innodb/trx_mutex                       | 780820227835076 | NULL                                                         | lock           |
|        34 |     1417 | wait/synch/mutex/innodb/redo_rseg_mutex                 | 780820229338204 | NULL                                                         | lock           |
|        34 |     1418 | wait/synch/mutex/innodb/redo_rseg_mutex                 | 780820230114012 | NULL                                                         | lock           |
|        34 |     1419 | wait/synch/mutex/innodb/trx_mutex                       | 780820231970768 | NULL                                                         | lock           |
|        34 |     1420 | wait/synch/rwlock/sql/gtid_commit_rollback              | 780820234222952 | NULL                                                         | read_lock      |
|        34 |     1421 | wait/synch/mutex/sql/Gtid_state                         | 780820236728444 | NULL                                                         | lock           |
|        34 |     1422 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_xids           | 780820244790828 | NULL                                                         | lock           |
|        34 |     1423 | wait/synch/rwlock/sql/Trans_delegate::lock              | 780820246715300 | NULL                                                         | read_lock      |
|        34 |     1424 | wait/synch/mutex/sql/LOCK_plugin                        | 780820247389952 | NULL                                                         | lock           |
|        34 |     1425 | wait/synch/mutex/sql/LOCK_plugin                        | 780820248388136 | NULL                                                         | lock           |
|        34 |     1426 | wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_done           | 780820249142208 | NULL                                                         | lock           |
|        34 |     1428 | wait/synch/mutex/innodb/trx_mutex                       | 780820270920008 | NULL                                                         | lock           |
|        34 |     1429 | wait/synch/mutex/sql/THD::LOCK_thd_data                 | 780820272200760 | NULL                                                         | lock           |
|        34 |     1430 | wait/synch/mutex/innodb/trx_mutex                       | 780820272818564 | NULL                                                         | lock           |
|        34 |     1431 | wait/synch/mutex/innodb/trx_mutex                       | 780820273066856 | NULL                                                         | lock           |
|        34 |     1432 | wait/synch/mutex/sql/LOCK_table_cache                   | 780820276659148 | NULL                                                         | lock           |
|        34 |     1434 | wait/io/socket/sql/client_connection                    | 780820292807324 | :0                                                           | send           |
|        34 |     1436 | wait/synch/mutex/sql/THD::LOCK_thd_query                | 780820339364164 | NULL                                                         | lock           |
|        34 |     1437 | wait/synch/mutex/sql/THD::LOCK_thd_query                | 780820361394436 | NULL                                                         | lock           |
+-----------+----------+---------------------------------------------------------+-----------------+--------------------------------------------------------------+----------------+
183 rows in set (0.00 sec)
```
