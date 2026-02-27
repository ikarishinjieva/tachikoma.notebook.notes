---
title: 20231218 - 在Oceanbase上, 开发MySQL的audit log
confluence_page_id: 2589916
created_at: 2023-12-18T16:08:48+00:00
updated_at: 2023-12-26T02:22:27+00:00
---

# 寻找可模仿的内部表

文件: ob_inner_table_schema_def.py

表: 

  - __all_ddl_operation
  - __all_virtual_sql_audit
  - __all_ddl_error_message
  - __all_tenant_security_audit_record
  - __all_virtual_sql_audit
  - DBA_AUDIT_TRAIL
  - __all_virtual_security_audit
  - __all_virtual_security_audit_history
  - AUDIT_ACTIONS
  - __all_server_event_history
  - GV$OB_SQL_AUDIT
  - V$OB_SQL_AUDIT

# __all_ddl_operation

租户表的结构初始化: 

```
//ob_inner_table_schema_def.py
 
def_table_schema(
    owner = 'yanmu.ztl',
    table_name    = '__all_ddl_operation',
    table_id      = '5',
    table_type = 'SYSTEM_TABLE',
    gm_columns = ['gmt_create', 'gmt_modified'],
    rowkey_columns = [
        ('schema_version', 'int'),
    ],
    in_tenant_space = True,

    normal_columns = [
      ('tenant_id', 'int'),
      ('user_id', 'int'),
      ('database_id', 'int'),
      ('database_name', 'varchar:OB_MAX_DATABASE_NAME_LENGTH'),
      ('tablegroup_id', 'int'),
      ('table_id', 'int'),
      ('table_name', 'varchar:OB_MAX_CORE_TALBE_NAME_LENGTH'),
      ('operation_type', 'int'),
      ('ddl_stmt_str', 'longtext'),
      ('exec_tenant_id', 'int', 'false', '1'),
    ],

)
``` 

索引初始化: 

```
//ob_inner_table_schema_def.py
 
def_sys_index_table(
  index_name = 'idx_ddl_type',
  index_table_id = 100006,
  index_columns = ['operation_type', 'schema_version'],
  index_using_type = 'USING_BTREE',
  index_type = 'INDEX_TYPE_NORMAL_LOCAL',
  keywords = all_def_keywords['__all_ddl_operation'])
``` 

全局表的结构初始化: 

```
//ob_inner_table_schema_def.py
 
def_table_schema(**gen_iterate_virtual_table_def(
  table_id = '12059',
  table_name = '__all_virtual_ddl_operation',
  keywords = all_def_keywords['__all_ddl_operation']))
``` 

表结构代码定义: 

```
int ObInnerTableSchema::all_ddl_operation_schema(ObTableSchema &table_schema)
{
  int ret = OB_SUCCESS;
  uint64_t column_id = OB_APP_MIN_COLUMN_ID - 1;

  //generated fields:
  table_schema.set_tenant_id(OB_SYS_TENANT_ID);
  table_schema.set_tablegroup_id(OB_SYS_TABLEGROUP_ID);
  table_schema.set_database_id(OB_SYS_DATABASE_ID);
  table_schema.set_table_id(OB_ALL_DDL_OPERATION_TID);
  table_schema.set_rowkey_split_pos(0);
  table_schema.set_is_use_bloomfilter(false);
  table_schema.set_progressive_merge_num(0);
  table_schema.set_rowkey_column_num(1);
  table_schema.set_load_type(TABLE_LOAD_TYPE_IN_DISK);
  table_schema.set_table_type(SYSTEM_TABLE);
  table_schema.set_index_type(INDEX_TYPE_IS_NOT);
  table_schema.set_def_type(TABLE_DEF_TYPE_INTERNAL);
 
...
}
``` 

调用表结构代码: 

```
//ob_inner_table_schema.h
 
const schema_create_func core_table_schema_creators [] = {
  ObInnerTableSchema::all_table_schema,
  ObInnerTableSchema::all_column_schema,
  ObInnerTableSchema::all_ddl_operation_schema,
  NULL,};
``` 

插入记录: 

```
//ob_ddl_sql_service.cpp
 
// sql_exec_tenant_id is valid, indicating that the __all_ddl_operation of the corresponding tenant needs to be written
int ObDDLSqlService::log_operation(
  ObSchemaOperation &schema_operation,
  common::ObISQLClient &sql_client,
  const int64_t sql_exec_tenant_id /*= OB_INVALID_TENANT_ID*/,
  common::ObSqlString *public_sql_string /*= NULL*/) {
...
 
    if (OB_FAIL(sql_string->append_fmt("INSERT INTO %s (SCHEMA_VERSION, TENANT_ID, EXEC_TENANT_ID, USER_ID, DATABASE_ID, "
                "DATABASE_NAME, TABLEGROUP_ID, TABLE_ID, TABLE_NAME, OPERATION_TYPE, DDL_STMT_STR, gmt_modified) "
                "values (%ld, %lu, %lu, %lu, %lu, '%.*s', %ld, %lu, '%.*s', %d, %.*s, now(6))",
                OB_ALL_DDL_OPERATION_TNAME,
                schema_operation.schema_version_,
                is_tenant_operation(schema_operation.op_type_) ? schema_operation.tenant_id_ : OB_INVALID_TENANT_ID,
                static_cast<int64_t>(exec_tenant_id), // not used after schema splited
                fill_schema_id(sql_tenant_id, schema_operation.user_id_),
                fill_schema_id(sql_tenant_id, schema_operation.database_id_),
                schema_operation.database_name_.length(),
                schema_operation.database_name_.ptr(),
                fill_schema_id(sql_tenant_id, schema_operation.tablegroup_id_),
                fill_schema_id(sql_tenant_id, schema_operation.table_id_),
                schema_operation.table_name_.length(),
                schema_operation.table_name_.ptr(),
                schema_operation.op_type_,
                ddl_str_hex.length(),
                ddl_str_hex.ptr()))) {
      LOG_WARN("sql string append format string failed, ", K(ret));
 
...
}
``` 

插入记录的堆栈: 

```
Thread 41 "DDLQueueTh0" hit Breakpoint 1, oceanbase::share::schema::ObDDLSqlService::log_operation (
    this=0x7f4d0bdff950, schema_operation=..., sql_client=..., sql_exec_tenant_id=0, public_sql_string=0x0)
    at ./src/share/schema/ob_ddl_sql_service.cpp:37
37	  int ret = OB_SUCCESS;
 
(gdb) bt
#0  oceanbase::share::schema::ObDDLSqlService::log_operation (this=0x7f4d0bdff950, schema_operation=...,
    sql_client=..., sql_exec_tenant_id=0, public_sql_string=0x0) at ./src/share/schema/ob_ddl_sql_service.cpp:37
#1  0x0000560a513b0756 in oceanbase::share::schema::ObTableSqlService::log_operation_wrapper (this=0x7f4d0bdff950,
    opt=..., sql_client=...) at ./src/share/schema/ob_table_sql_service.cpp:4144
#2  0x0000560a513dcda7 in oceanbase::share::schema::ObTableSqlService::create_table (this=0x7f4d0bdff950,
    table=..., sql_client=..., ddl_stmt_str=0x7f4cead4d100, need_sync_schema_version=true, is_truncate_table=false)
    at ./src/share/schema/ob_table_sql_service.cpp:2404
#3  0x0000560a46d05823 in oceanbase::rootserver::ObDDLOperator::create_table (this=0x7f4cead4d4b8,
    table_schema=..., trans=..., ddl_stmt_str=0x7f4cead4d100, need_sync_schema_version=true,
    is_truncate_table=false) at ./src/rootserver/ob_ddl_operator.cpp:1479
#4  0x0000560a471273bb in oceanbase::rootserver::ObDDLService::create_tables_in_trans (
    this=0x560a558a3800 <oceanbase::observer::ObServer::get_instance()::THE_ONE+4422336>, if_not_exist=false,
    ddl_stmt_str=..., error_info=..., table_schemas=..., sequence_ddl_arg=..., last_replay_log_id=0,
    dep_infos=0x7f4c614057c0, mock_fk_parent_table_schema_array=...) at ./src/rootserver/ob_ddl_service.cpp:1815
#5  0x0000560a4710bb83 in oceanbase::rootserver::ObDDLService::create_user_tables (
    this=0x560a558a3800 <oceanbase::observer::ObServer::get_instance()::THE_ONE+4422336>, if_not_exist=false,
    ddl_stmt_str=..., error_info=..., table_schemas=..., schema_guard=..., sequence_ddl_arg=...,
    last_replay_log_id=0, dep_infos=0x7f4c614057c0, mock_fk_parent_table_schema_array=...)
    at ./src/rootserver/ob_ddl_service.cpp:386
#6  0x0000560a476751ab in oceanbase::rootserver::ObRootService::create_table (
    this=0x560a55589b00 <oceanbase::observer::ObServer::get_instance()::THE_ONE+1170880>, arg=..., res=...)
    at ./src/rootserver/ob_root_service.cpp:3439
#7  0x0000560a48c83b52 in oceanbase::rootserver::ObRpcCreateTableP::leader_process (this=0x7f4c61404060)
    at ./src/rootserver/ob_rs_rpc_processor.h:334
#8  0x0000560a48c647de in oceanbase::rootserver::ObRootServerRPCProcessorBase::process_ (this=0x7f4c61405830,
    pcode=oceanbase::obrpc::OB_CREATE_TABLE) at ./src/rootserver/ob_rs_rpc_processor.h:200
#9  0x0000560a48c839e3 in oceanbase::rootserver::ObRootServerRPCProcessor<(oceanbase::obrpc::ObRpcPacketCode)515>::process (this=0x7f4c61404060) at ./src/rootserver/ob_rs_rpc_processor.h:262
#10 0x0000560a45b40220 in oceanbase::obrpc::ObRpcProcessorBase::run (this=0x7f4c61404060)
    at ./deps/oblib/src/rpc/obrpc/ob_rpc_processor_base.cpp:82
#11 0x0000560a533b65b0 in oceanbase::rpc::frame::ObReqQHandler::handlePacketQueue (
    this=0x560a55554080 <oceanbase::observer::ObServer::get_instance()::THE_ONE+951104>, req=0x7f4ce5998818)
    at ./deps/oblib/src/rpc/frame/ob_req_qhandler.cpp:79
#12 0x0000560a533b7395 in oceanbase::rpc::frame::ObReqQueue::process_task (this=0x7f4d0e1ffc50,
    task=0x7f4ce5998818) at ./deps/oblib/src/rpc/frame/ob_req_queue_thread.cpp:143
#13 0x0000560a45c24198 in oceanbase::rpc::frame::ObReqQueue::loop (this=0x7f4d0e1ffc50)
    at ./deps/oblib/src/rpc/frame/ob_req_queue_thread.cpp:170
#14 0x0000560a4844a839 in oceanbase::observer::QueueThread::Thread::run1 (this=0x7f4d0e1ffc10)
    at ./src/observer/ob_srv_deliver.h:79
#15 0x0000560a50acbe43 in oceanbase::share::MyObThreadPool::run1 (this=0x7f4d14cf9790)
    at ./src/share/ob_thread_mgr.h:47
#16 0x0000560a52a831c3 in oceanbase::lib::Threads::run (this=0x7f4d14cf9790, idx=0)
    at ./deps/oblib/src/lib/thread/threads.cpp:205
#17 0x0000560a52a7f7c8 in oceanbase::lib::Thread::run (this=0x7f4d0e1ffe10)
    at ./deps/oblib/src/lib/thread/thread.cpp:164
#18 0x0000560a52a7f2de in oceanbase::lib::Thread::__th_start (arg=0x7f4d0e1ffe10)
    at ./deps/oblib/src/lib/thread/thread.cpp:311
#19 0x00007f4d153b56db in start_thread (arg=0x7f4cead7d300) at pthread_create.c:463
#20 0x00007f4d150de61f in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
``` 

结合slow log的堆栈, 分析出应该将代码点加入哪里

# 查询slow log相关的堆栈

相关参数: trace_log_slow_query_watermark

  - 点位1: ObMPQuery::process (处理COM_QUERY) -> process_single_stmt

发现一个额外的有用的类: ObSQLUtils::handle_audit_record, 需调查

# 调查ObSQLUtils::handle_audit_record

  - ObSQLUtils::handle_audit_record
    - ObMySQLRequestManager::record_request 
      - (记录会保存到ObMySQLRequestManager._queue中)
      - (记录使用: ObGvSqlAudit::inner_get_next_row)

此处设计: 内部表使用了内存结构

我们先尝试使用磁盘结构的内部表

# 使用磁盘结构的内部表

  - ObMPConnect::unlock_user_if_time_is_up (自主创建事务)

# DEMO

往如下表中插入数据: 

```
def_table_schema(
  owner = 'linlin.xll',
  table_name     = '__all_job_log',
  table_id       = '325',
  table_type     = 'SYSTEM_TABLE',
  gm_columns     = ['gmt_create', 'gmt_modified'],
  rowkey_columns = [
    ('tenant_id', 'int'),
    ('job', 'int', 'false'),
    ('time', 'timestamp', 'false'),
    ('exec_addr', 'varchar:MAX_IP_PORT_LENGTH', 'false')
  ],
  in_tenant_space = True,
  normal_columns = [
    ('code', 'int', 'true', '0'),
    ('message', 'varchar:4000')
  ],
)

``` 

插入数据的代码位置: ObSQLUtils::handle_audit_record

模仿逻辑: ObMPConnect::unlock_user_if_time_is_up

  - 手工开启事务
  - 向目标表插入数据
  - 手工提交事务

试验成功
