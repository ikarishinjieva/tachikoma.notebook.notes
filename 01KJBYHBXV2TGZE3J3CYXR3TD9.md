---
title: 20220213 - 浦发MGR切换问题诊断
confluence_page_id: 1736885
created_at: 2022-02-23T10:59:52+00:00
updated_at: 2022-03-02T09:00:01+00:00
---

# 报错

```
2022-01-15T16:01:51.017651+08:00 10 [ERROR] [MY-011708] [Repl] Plugin group_replication reported: 'There was an error when trying to access the server with user: mysql.session. Make sure the user is present in the server and that the MySQL upgrade procedure was run correctly.'
2022-01-15T16:01:51.018013+08:00 10 [ERROR] [MY-011564] [Repl] Plugin group_replication reported: 'Failed to establish an internal server connection to execute plugin operations'
2022-01-15T16:01:51.019680+08:00 10 [ERROR] [MY-011560] [Repl] Plugin group_replication reported: 'Error when contacting the server to ensure the proper logging of a group change in the binlog'
2022-01-15T16:01:51.019939+08:00 10 [ERROR] [MY-011445] [Repl] Plugin group_replication reported: 'Error at event handling! Got error: 1.'
2022-01-15T16:01:51.020211+08:00 10 [ERROR] [MY-011452] [Repl] Plugin group_replication reported: 'Fatal error during execution on the Applier process of Group Replication. The server will now leave the group.'
2022-01-15T16:01:51.055068+08:00 10 [Warning] [MY-011630] [Repl] Plugin group_replication reported: 'Due to a plugin error, some transactions were unable to be certified and will now rollback.'
``` 

对其场景进行分析

根据错误信息, 找到报错函数 Sql_service_interface::set_session_user

断点设置在 Sql_service_interface::set_session_user, 获取堆栈

堆栈之一: 

```
(gdb) bt
#0  0x00007f6a71923e70 in Sql_service_interface::set_session_user(char const*)@plt ()
   from /root/opt/mysql/8.0.21/lib/plugin/group_replication.so
#1  0x00007f6a719e5d1c in Sql_service_command_interface::establish_session_connection (
    this=this@entry=0x7f6a0000dac0, isolation_param=isolation_param@entry=PSESSION_USE_THREAD,
    user=user@entry=0x7f6a71a8228d "mysql.session", plugin_pointer=plugin_pointer@entry=0x0)
    at ../../../mysql-8.0.21/plugin/group_replication/src/sql_service/sql_service_command.cc:58
#2  0x00007f6a7194f683 in Certifier::initialize_server_gtid_set (this=0x7f6a28050a90,
    get_server_gtid_retrieved=<optimized out>)
    at ../../../mysql-8.0.21/plugin/group_replication/src/certifier.cc:351
#3  0x00007f6a71950324 in Certifier::initialize (this=0x7f6a28050a90,
    gtid_assignment_block_size=1000000)
    at ../../../mysql-8.0.21/plugin/group_replication/src/certifier.cc:559
#4  0x00007f6a7197994d in Certification_handler::handle_action (this=0x7f6a28088a30,
    action=0x7f6a000191e0)
    at ../../../mysql-8.0.21/plugin/group_replication/src/handlers/certification_handler.cc:88
#5  0x00007f6a71940ea8 in Applier_module::setup_pipeline_handlers (this=0x7f6a280090b0)
    at ../../../mysql-8.0.21/plugin/group_replication/src/applier.cc:184
#6  0x00007f6a71943070 in Applier_module::applier_thread_handle (this=0x7f6a280090b0)
    at ../../../mysql-8.0.21/plugin/group_replication/src/applier.cc:412
#7  0x00007f6a71944649 in launch_handler_thread (arg=arg@entry=0x7f6a280090b0)
    at ../../../mysql-8.0.21/plugin/group_replication/src/applier.cc:50
#8  0x000000000245661c in pfs_spawn_thread (arg=0x7f6a30218bb0)
    at ../../../mysql-8.0.21/storage/perfschema/pfs.cc:2880
#9  0x00007f6a9fabf6db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#10 0x00007f6a9da8671f in clone () from /lib/x86_64-linux-gnu/libc.so.6
``` 

核心问题: session connection是哪个方向的, 是主动端还是被动端在报错
