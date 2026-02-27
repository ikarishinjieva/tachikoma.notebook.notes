---
title: 20211125 - 滨银汽车MySQL crash
confluence_page_id: 1573166
created_at: 2021-11-25T03:28:21+00:00
updated_at: 2021-12-19T16:15:03+00:00
---

# 问题

数据库版本: mysql-8.0.11-linux-glibc2.12-x86_64.tar

```
2021-11-23T09:28:46.568699+08:00 5 [Note] [MY-010559] [Repl] Multi-threaded slave statistics for channel '': seconds elapsed = 147; events assigned = 74067969; worker queues filled over overrun level = 286007; waited due a Worker queue full = 9269; waited due the total size = 61955; waited at clock conflicts = 10394708676700 waited (count) when Workers occupied = 10796515 waited when Workers occupied = 243523025600 
stack_bottom = 7f6e380a1d80 thread_stack 0x46000 
/usr/local/mysql/bin/mysqld(my_print_stacktrace(unsigned char*, unsigned long)+0x2e) [0x19fcc8e] 
/usr/local/mysql/bin/mysqld(handle_fatal_signal+0x413) [0xccc863] 
/lib64/libpthread.so.0(+0xf630) [0x7f7137d0e630] 
/usr/local/mysql/bin/mysqld(Copy_field::invoke_do_copy2(Copy_field*)+0x1e) [0xd9dd1e] 
/usr/local/mysql/bin/mysqld(Copy_field::invoke_do_copy(Copy_field*)+0x15) [0xd9dcb5] 
/usr/local/mysql/bin/mysqld(store_key_field::copy_inner()+0x18) [0xc0c548] 
/usr/local/mysql/bin/mysqld(cp_buffer_from_ref(THD*, TABLE*, TABLE_REF*)+0x8c) [0xb977bc] 
/usr/local/mysql/bin/mysqld(get_quick_select_for_ref(THD*, TABLE*, TABLE_REF*, unsigned long long)+0xc9) [0xb26b19] /usr/local/mysql/bin/mysqld(JOIN::add_sorting_to_table(unsigned int, ORDER_with_src*, bool)+0x185) [0xc13915] /usr/local/mysql/bin/mysqld(JOIN::make_tmp_tables_info()+0x2d0) [0xc13c90] 
/usr/local/mysql/bin/mysqld(JOIN::optimize()+0x20c6) [0xbc3a26] 
/usr/local/mysql/bin/mysqld(SELECT_LEX::optimize(THD*)+0x5f2) [0xc0ee32] 
/usr/local/mysql/bin/mysqld(SELECT_LEX_UNIT::optimize(THD*)+0x3b) [0xc6d1ab] 
/usr/local/mysql/bin/mysqld(SELECT_LEX::optimize(THD*)+0x641) [0xc0ee81] 
/usr/local/mysql/bin/mysqld(Sql_cmd_dml::execute_inner(THD*)+0xe4) [0xc0f324] 
/usr/local/mysql/bin/mysqld(Sql_cmd_dml::execute(THD*)+0x137) [0xc17547] 
/usr/local/mysql/bin/mysqld(mysql_execute_command(THD*, bool)+0x186f) [0xbcbf2f] 
/usr/local/mysql/bin/mysqld(mysql_parse(THD*, Parser_state*)+0x30f) [0xbd040f] 
/usr/local/mysql/bin/mysqld(dispatch_command(THD*, COM_DATA const*, enum_server_command)+0x26ca) [0xbd2d3a] /usr/local/mysql/bin/mysqld(do_command(THD*)+0x179) [0xbd3be9] 
/usr/local/mysql/bin/mysqld() [0xcbfd80] 
/usr/local/mysql/bin/mysqld() [0x1aa5f5f] 
/lib64/libpthread.so.0(+0x7ea5) [0x7f7137d06ea5] 
/lib64/libc.so.6(clone+0x6d) [0x7f71361678dd]
``` 

# 分析

0xd9dd1e 对应的代码位: 

```
Breakpoint 1 at 0xd9dd1e: file ../../mysql-8.0.11/sql/field.h, line 852.
``` 

相关调用代码: 

  1. invoke_do_copy2 调用 check_and_set_temporary_null
     1. ![image2021-11-25 11:28:56.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-11-25%2011%3A28%3A56.png)
  2. check_and_set_temporary_null的代码如下
     1. ![image2021-11-25 11:29:12.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-11-25%2011%3A29%3A12.png)
     2. ![image2021-11-25 11:28:25.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-11-25%2011%3A28%3A25.png)
     3. check_and_set_temporary_null 的代码等价于: 

```
m_from_field && m_from_field.m_is_tmp_nullable && m_from_field.m_is_tmp_null && ...
``` 
  3. 查看代码崩溃位置:   
![image2021-11-25 12:30:7.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-11-25%2012%3A30%3A7.png)  

     1. 高亮三个框, 依次对应
        1. m_from_field == NULL
        2. m_from_field.m_is_tmp_nullable == 0  

           1. 验证 m_is_tmp_nullable 的偏移为 0x18
           2. ![image2021-11-25 12:32:40.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-11-25%2012%3A32%3A40.png)
        3. m_from_field.m_is_tmp_null == 0
           1. 验证 m_is_tmp_nullable 的偏移为 0x19
           2. ![image2021-11-25 12:33:29.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-11-25%2012%3A33%3A29.png)
     2. 崩溃位置 0xd9dd1e, 对应崩溃原因为
        1. m_from_field != 0
        2. m_from_field.m_is_tmp_nullable 为非法地址
     3. 以此判断: m_from_field 被其他内存泄漏污染, 导致变为非法地址

#   
建议

  1. 未在MySQL bug库中找到已有bug与之相关
  2. 建议开启coredump功能, 获取coredump后, 可能能观察到内存泄漏的原因
  3. 开启general log, 排查哪个SQL与崩溃相关, 如果测试环境可以复现, 有利于排查
  4. 可以先尝试升级到MySQL最新版本, 问题可能已经修复

# 进展

已获取coredump 进行分析: <https://support.actionsky.com/service_desk/browse/BEIJ-1990>

另: 用户测试crash与以下SQL相关: 

```
 
``` 

崩溃堆栈

```
(gdb) bt
#0  0x00007f3a3cf14aa1 in pthread_kill () from /lib64/libpthread.so.0
#1  0x00000000019fc317 in my_write_core (sig=<optimized out>) at ../../mysql-8.0.11/mysys/stacktrace.cc:278
#2  0x0000000000ccc895 in handle_fatal_signal (sig=11) at ../../mysql-8.0.11/sql/signal_handler.cc:249
#3  <signal handler called>
#4  Field::is_null (this=0x0, row_offset=row_offset@entry=0) at ../../mysql-8.0.11/sql/field.cc:10217
#5  0x0000000000c0c553 in store_key_field::copy_inner (this=0x7f36e81ddb28) at ../../mysql-8.0.11/sql/sql_select.h:982
#6  0x0000000000b977bc in copy (this=<optimized out>) at ../../mysql-8.0.11/sql/sql_select.h:919
#7  cp_buffer_from_ref (thd=thd@entry=0x7f36e80bff40, table=table@entry=0x7f36e815de90, ref=ref@entry=0x7f36e81dcb48)
    at ../../mysql-8.0.11/sql/sql_executor.cc:5656
#8  0x0000000000b26b19 in get_quick_select_for_ref (thd=0x7f36e80bff40, table=0x7f36e815de90, ref=0x7f36e81dcb48, records=18635)
    at ../../mysql-8.0.11/sql/opt_range.cc:10248
#9  0x0000000000c13915 in JOIN::add_sorting_to_table (this=this@entry=0x7f36e81dc600, idx=idx@entry=0,
    sort_order=sort_order@entry=0x7f372d9d8ca0, force_stable_sort=force_stable_sort@entry=false)
    at ../../mysql-8.0.11/sql/sql_select.cc:4393
#10 0x0000000000c13c90 in JOIN::make_tmp_tables_info (this=this@entry=0x7f36e81dc600) at ../../mysql-8.0.11/sql/sql_select.cc:4136
#11 0x0000000000bc3a26 in JOIN::optimize (this=0x7f36e81dc600) at ../../mysql-8.0.11/sql/sql_optimizer.cc:722
#12 0x0000000000c0ee32 in SELECT_LEX::optimize (this=this@entry=0x7f36e8150cf8, thd=thd@entry=0x7f36e80bff40)
    at ../../mysql-8.0.11/sql/sql_select.cc:1488
#13 0x0000000000c6d1ab in SELECT_LEX_UNIT::optimize (this=this@entry=0x7f36e8150ff0, thd=thd@entry=0x7f36e80bff40)
    at ../../mysql-8.0.11/sql/sql_union.cc:752
#14 0x0000000000c0ee81 in SELECT_LEX::optimize (this=<optimized out>, thd=0x7f36e80bff40) at ../../mysql-8.0.11/sql/sql_select.cc:1495
#15 0x0000000000c0f324 in Sql_cmd_dml::execute_inner (this=0x7f36e802ee00, thd=0x7f36e80bff40)
    at ../../mysql-8.0.11/sql/sql_select.cc:640
#16 0x0000000000c17547 in Sql_cmd_dml::execute (this=0x7f36e802ee00, thd=0x7f36e80bff40) at ../../mysql-8.0.11/sql/sql_select.cc:554
#17 0x0000000000bcbf2f in mysql_execute_command (thd=thd@entry=0x7f36e80bff40, first_level=first_level@entry=true)
    at ../../mysql-8.0.11/sql/sql_parse.cc:4220
#18 0x0000000000bd040f in mysql_parse (thd=thd@entry=0x7f36e80bff40, parser_state=parser_state@entry=0x7f372d9da600)
    at ../../mysql-8.0.11/sql/sql_parse.cc:4935
#19 0x0000000000bd2d3a in dispatch_command (thd=thd@entry=0x7f36e80bff40, com_data=com_data@entry=0x7f372d9dad00,
    command=<optimized out>) at ../../mysql-8.0.11/sql/sql_parse.cc:1589
#20 0x0000000000bd3be9 in do_command (thd=thd@entry=0x7f36e80bff40) at ../../mysql-8.0.11/sql/sql_parse.cc:1214
#21 0x0000000000cbfd80 in handle_connection (arg=arg@entry=0x517ed60)
    at ../../mysql-8.0.11/sql/conn_handler/connection_handler_per_thread.cc:308
#22 0x0000000001aa5f5f in pfs_spawn_thread (arg=0x536c7c0) at ../../../mysql-8.0.11/storage/perfschema/pfs.cc:2814
#23 0x00007f3a3cf0fea5 in start_thread () from /lib64/libpthread.so.0
#24 0x00007f3a3b3708dd in __libc_ifunc_impl_list () from /lib64/libc.so.6
#25 0x0000000000000000 in ?? ()
``` 

看到frame 4的this为空, 即以下代码中to_field为空: 

![image2021-12-9 14:54:8.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-9%2014%3A54%3A8.png)

查看frame 5的 copy_field的值: 

![image2021-12-9 14:54:28.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-9%2014%3A54%3A28.png)

发现其内容都为空

往上扩展, copy_field的拥有者的内容也都为空: 

![image2021-12-9 14:56:9.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-9%2014%3A56%3A9.png)

相关代码: 

![image2021-12-9 15:5:41.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-9%2015%3A5%3A41.png)

继续向上找到父结构: 

![image2021-12-9 15:6:0.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-9%2015%3A6%3A0.png)

发现key_copy的内容是空

查看key_copy的相关内存: 

![image2021-12-9 15:7:38.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-9%2015%3A7%3A38.png)

发现其附近内存都被清零, 从0x7f36e81dd9f8有部分数据, 然后清零.

猜测是0x7f36e81dd9f8对应的变量有内存泄漏

可以看到 0x7f36e81dd9f8 是ref->key_buff2, 有可能是key_buff或者key_buff2的泄露

![image2021-12-9 15:6:0.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-9%2015%3A6%3A0.png)

# 进展

在测试环境上, 新建create database, 然后按用户提供的SQL, create table, 然后select, 有大概率触发崩溃

相关SQL: 

```
  CREATE TABLE `afp_record_new` (
  `id` varchar(64) NOT NULL,
  `afp_id` varchar(64) DEFAULT NULL COMMENT '案件ID',
  `mission_id` varchar(64) DEFAULT NULL COMMENT '任务ID',
  `linkman` varchar(64) DEFAULT NULL COMMENT '联系人',
  `link_infomation` varchar(64) DEFAULT NULL COMMENT '联系方式',
  `afp_sign` varchar(64) DEFAULT NULL COMMENT '催收代码',
  `afp_result` varchar(64) DEFAULT NULL COMMENT '催收结果',
  `appointment_time` varchar(64) DEFAULT NULL COMMENT '约会时间',
  `allowance` varchar(64) DEFAULT NULL COMMENT '承若金额',
  `afp_date` varchar(64) DEFAULT NULL COMMENT '催收日期',
  `afp_record` varchar(4000) DEFAULT NULL COMMENT '催收备注',
  `all_date` varchar(64) DEFAULT NULL COMMENT '承若日期',
  `user_id` varchar(64) DEFAULT NULL COMMENT '处理人',
  `real_user` varchar(64) DEFAULT NULL COMMENT '实际处理人',
  `escrow_user` varchar(64) DEFAULT NULL COMMENT '代管人',
  `station` varchar(64) DEFAULT NULL COMMENT '岗位',
  `close_date` varchar(64) DEFAULT NULL COMMENT '关闭日期',
  `act_sign` varchar(64) DEFAULT NULL COMMENT '行动代码',
  `act_sign_notes` varchar(64) DEFAULT NULL COMMENT '行动代码注释',
  `input_time` varchar(64) DEFAULT NULL COMMENT '录入时间',
  `import_data` varchar(64) DEFAULT NULL COMMENT '是否为导入数据',
  `afp_sign_explain` varchar(255) DEFAULT NULL COMMENT '催收代码的中文解释',
  `all_sign` varchar(255) DEFAULT NULL COMMENT '承诺兑现标识',
  `input_user` varchar(255) DEFAULT NULL COMMENT '录入人',
  PRIMARY KEY (`id`),
  KEY `idx_afp_record_new_1` (`afp_id`),
  KEY `idx_afp_record_new_2` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='最新催收记录表 ^ 记录每个案件的最新催收记录，表结构同afp_record' ;

CREATE TABLE `mission_info` (
  `id` varchar(64) NOT NULL,
  `queue_id` varchar(64) DEFAULT NULL COMMENT '队列ID',
  `afp_id` varchar(64) DEFAULT NULL COMMENT '催收案件ID',
  `user_id` varchar(64) DEFAULT NULL COMMENT '催收人ID',
  `parent_user_id` varchar(64) DEFAULT NULL COMMENT '组长',
  `parent_mission_id` varchar(64) DEFAULT NULL COMMENT '任务父级ID',
  `help_mission_flag` varchar(64) DEFAULT NULL COMMENT '协助标志',
  `lock_flag` varchar(64) DEFAULT NULL COMMENT '锁定标志',
  `status` varchar(64) DEFAULT NULL COMMENT '状态',
  `create_time` varchar(64) DEFAULT NULL COMMENT '创建日期',
  `update_time` varchar(64) DEFAULT NULL COMMENT '更新日期',
  `create_user` varchar(64) DEFAULT NULL COMMENT '创建人',
  `update_user` varchar(64) DEFAULT NULL COMMENT '更新人',
  `complete_time` varchar(64) DEFAULT NULL COMMENT '完成时间',
  `escrow_user` varchar(64) DEFAULT NULL COMMENT '代管人',
  `escrow_time` varchar(64) DEFAULT NULL COMMENT '代管截止日期',
  `handle` varchar(255) DEFAULT NULL COMMENT '是否处理',
  `co_status` varchar(32) DEFAULT NULL COMMENT '协办状态',
  `turn_status` varchar(32) DEFAULT NULL COMMENT '转队列状态',
  `leave_status` varchar(32) DEFAULT NULL COMMENT '留案状态',
  `co_time` varchar(500) DEFAULT NULL COMMENT '协办到期日',
  `entrust_money` varchar(64) DEFAULT NULL COMMENT '外包委托金额',
  `will_send_company` varchar(64) DEFAULT NULL COMMENT '预派案公司',
  `send_case_date` varchar(64) DEFAULT NULL COMMENT '派单时间',
  `dead_time_of_the_send_case` date DEFAULT NULL COMMENT '派案到期日',
  `entrust_over_due_days` varchar(64) DEFAULT NULL COMMENT '委托逾期天数',
  `send_approve_status` varchar(2) DEFAULT NULL COMMENT '外包案件审批状态 0 未审批 1 主管审批通过 2 科长审批通过 3 部长审批通过',
  `overdue_receivables` varchar(64) DEFAULT NULL COMMENT '逾期应收款总计',
  `residual_amount` varchar(64) DEFAULT NULL COMMENT '剩余本金总额',
  `overdue_total` varchar(64) DEFAULT NULL,
  `back_case_status` varchar(32) DEFAULT NULL COMMENT '退案状态',
  `leave_time` varchar(64) DEFAULT NULL COMMENT '留案时间',
  `delay_time` varchar(64) DEFAULT NULL,
  `co_user` varchar(500) DEFAULT NULL COMMENT '协办人',
  `turn_user` varchar(64) DEFAULT NULL COMMENT '转队列人员',
  `co_queue_id` varchar(500) DEFAULT NULL COMMENT '协办到队列ID',
  `history_co_queue_id` varchar(500) DEFAULT NULL COMMENT '历史协办队列',
  `history_co_user` varchar(500) DEFAULT NULL COMMENT '历史协办人',
  `history_co_time` varchar(500) DEFAULT NULL COMMENT '历史协办到期日',
  `send_case_news_flag` varchar(10) DEFAULT NULL COMMENT '派案申请消息提醒标识 null：不需提醒 0：未读取 1:已读取 ^ 最后一级审批通过时，进行消息提醒，=1时已读取，不在提醒',
  `news_flag` varchar(10) DEFAULT NULL COMMENT '新增案件消息提醒标识 null：不需提醒 0：未读取（需提醒）  1:已读取(已提醒） ^ 外包及法诉队列需要提醒，其他队列不需要',
  `history_co_news_flag` varchar(100) DEFAULT NULL COMMENT '历史协办消息提醒标识 null：不需提醒 0：未读取（需提醒）  1:已读取(已提醒） ^ 外包及法诉队列需要提醒，其他队列不需要',
  `write_off_flag` varchar(2) DEFAULT NULL COMMENT '核销队列标识 ^ null-未核销 1-CMS核销后处理队列 2-核销后不追偿队列',
  `car_type` tinyint(2) DEFAULT NULL COMMENT '品牌类别 ^ t_dict.type=''car_type''',
  `redistribution_flag` char(1) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '重分配标识',
  `visiting_logo` int(11) NOT NULL COMMENT '外访标识',
  PRIMARY KEY (`id`),
  KEY `afp_id` (`afp_id`) USING BTREE,
  KEY `idx_mission_info_queue_id` (`queue_id`),
  KEY `idx_mission_info_car_type` (`car_type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='任务表' ;

CREATE TABLE `sys_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL COMMENT '登录名',
  `password` varchar(32) NOT NULL COMMENT '密码',
  `salt` varchar(32) NOT NULL,
  `nickname` varchar(255) DEFAULT NULL,
  `headImgUrl` varchar(255) DEFAULT NULL,
  `phone` varchar(11) DEFAULT NULL,
  `telephone` varchar(30) DEFAULT NULL,
  `email` varchar(50) DEFAULT NULL,
  `birthday` date DEFAULT NULL,
  `sex` tinyint(1) DEFAULT NULL,
  `status` tinyint(1) NOT NULL DEFAULT '1',
  `createTime` datetime NOT NULL,
  `updateTime` datetime NOT NULL,
  `position_id` varchar(255) DEFAULT NULL COMMENT '职位',
  `is_os` varchar(2) DEFAULT '0' COMMENT '是否是外包队列 0 不是  1 是',
  `widthdraw_money_model` varchar(2) DEFAULT NULL COMMENT '收款模式',
  `widthdraw_car_model` varchar(2) DEFAULT NULL COMMENT '控车模式',
  `pwd_update_date` date DEFAULT NULL COMMENT '密码修改日期',
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1222637 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户表' ;

CREATE TABLE `customer_basic` (
  `id` int(64) NOT NULL AUTO_INCREMENT,
  `afp_id` varchar(64) DEFAULT '' COMMENT '合同id',
  `name` varchar(32) DEFAULT '' COMMENT '姓名',
  `role_name` varchar(16) DEFAULT '' COMMENT '角色',
  `five_type` varchar(4) DEFAULT '' COMMENT '五级分类',
  `occupation` varchar(32) DEFAULT '' COMMENT '职业',
  `unit_name` varchar(64) DEFAULT '' COMMENT '单位名称',
  `sex` varchar(16) DEFAULT '' COMMENT '性别',
  `document_type` varchar(64) DEFAULT '' COMMENT '证件类型',
  `document_num` varchar(32) DEFAULT '' COMMENT '证件号码',
  `birth_date` varchar(16) DEFAULT '' COMMENT '生日',
  `create_date` varchar(32) DEFAULT '' COMMENT '创建时间',
  `update_date` varchar(32) DEFAULT '' COMMENT '更新时间',
  `nation` varchar(32) DEFAULT '' COMMENT '民族',
  `province` varchar(16) DEFAULT '' COMMENT '省份',
  `city` varchar(64) DEFAULT '' COMMENT '城市',
  `counties` varchar(32) DEFAULT '' COMMENT '所在区县',
  `industry` varchar(16) DEFAULT '' COMMENT '所在行业',
  `objective` varchar(16) DEFAULT '' COMMENT '购车目的',
  `home_phone` varchar(16) DEFAULT '' COMMENT '家庭电话',
  `office_phone` varchar(16) DEFAULT '' COMMENT '办公电话',
  `qq` varchar(16) DEFAULT '',
  `wechat` varchar(16) DEFAULT '' COMMENT '微信',
  `email` varchar(32) DEFAULT '',
  `hobby` varchar(16) DEFAULT '' COMMENT '客户爱好',
  `education` varchar(16) DEFAULT '' COMMENT '学历',
  `monthly_income` varchar(16) DEFAULT '' COMMENT '月收入',
  `position` varchar(16) DEFAULT '' COMMENT '职位',
  `family_structure` varchar(16) DEFAULT '' COMMENT '家庭结构',
  `person_charge` varchar(16) DEFAULT '' COMMENT '负责人',
  `person_charge_position` varchar(16) DEFAULT '' COMMENT '负责人职位',
  `person_charge_address` varchar(16) DEFAULT '' COMMENT '负责人地址',
  `pinyin` varchar(16) DEFAULT '' COMMENT '拼音',
  `marital_status` varchar(16) DEFAULT '' COMMENT '婚姻状态',
  PRIMARY KEY (`id`),
  KEY `afp_id` (`afp_id`) USING BTREE,
  KEY `role_name` (`role_name`) USING BTREE,
  KEY `idx_customer_basic_afp_id` (`afp_id`)
) ENGINE=InnoDB AUTO_INCREMENT=764002 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='客户基本信息表' ;

 CREATE TABLE `contract_info` (
  `id` varchar(64) NOT NULL,
  `afp_id` varchar(64) DEFAULT NULL COMMENT '催收案_id',
  `application_number` varchar(255) DEFAULT NULL COMMENT '合同号',
  `state` varchar(255) DEFAULT NULL COMMENT '合同状态',
  `loan_products` varchar(255) DEFAULT NULL COMMENT '贷款产品',
  `start_time` varchar(255) DEFAULT NULL COMMENT '合同起始日期',
  `end_time` varchar(255) DEFAULT NULL COMMENT '合同终止日期',
  `crete_time` varchar(255) DEFAULT NULL COMMENT '合同创建日期',
  `sign_time` varchar(255) DEFAULT NULL COMMENT '合同签署日期',
  `loan_amount` varchar(255) DEFAULT NULL COMMENT '贷款金额',
  `loan_num` varchar(255) DEFAULT NULL COMMENT '贷款期数',
  `first_ratio` varchar(255) DEFAULT NULL COMMENT '首付比例',
  `interst_rate` varchar(255) DEFAULT NULL COMMENT '利率',
  `loan_balance` varchar(255) DEFAULT NULL COMMENT '贷款余额',
  `paid_principal` varchar(255) DEFAULT NULL COMMENT '已偿还本金',
  `residual_amount` varchar(255) DEFAULT NULL COMMENT '剩余本金总额',
  `outstanding_principal` varchar(255) DEFAULT NULL COMMENT '未到期本金',
  `interest_due` varchar(255) DEFAULT NULL COMMENT '到期利息总计',
  `penalty` varchar(255) DEFAULT NULL COMMENT '罚息',
  `litigation_fee` varchar(255) DEFAULT NULL COMMENT '诉讼费',
  `collect_fare` varchar(255) DEFAULT NULL COMMENT '收车费',
  `without_fee` varchar(255) DEFAULT NULL COMMENT '未尝催收工本费',
  `latest_repayment` varchar(255) DEFAULT NULL COMMENT '最近一次还款金额',
  `latest_date` varchar(255) DEFAULT NULL COMMENT '最近一次还款日期',
  `state_verification` varchar(255) DEFAULT NULL COMMENT '核销状态',
  `overdue_days` varchar(255) DEFAULT NULL COMMENT '本次逾期天数',
  `number_overdue` varchar(255) DEFAULT NULL COMMENT '本期逾期天数',
  `overdue_receivables` varchar(255) DEFAULT NULL COMMENT '逾期应收款总计',
  `province` varchar(255) DEFAULT NULL COMMENT '省份',
  `city` varchar(255) DEFAULT NULL COMMENT '城市',
  `date_payment` varchar(255) DEFAULT NULL COMMENT '付款日',
  `loan_car` varchar(255) DEFAULT NULL COMMENT '贷款车型',
  `app_num` varchar(255) DEFAULT NULL COMMENT '申请号',
  `is_fixed` int(10) DEFAULT NULL COMMENT '是否是固定利率(1-是 0-否）',
  `vin_num` varchar(255) DEFAULT NULL COMMENT 'VIN 号',
  `overdue_money` varchar(255) DEFAULT NULL COMMENT '逾期费用',
  `engine_nbr` varchar(255) DEFAULT NULL COMMENT '发动机号',
  `payment_date` varchar(64) DEFAULT NULL COMMENT '付款日期',
  `remarks` varchar(255) DEFAULT NULL COMMENT '备注',
  `car_type` tinyint(2) DEFAULT NULL COMMENT '品牌类别 ^ t_dict.type=''car_type''',
  PRIMARY KEY (`id`),
  KEY `id` (`id`) USING BTREE,
  KEY `idx_contract_info_id` (`id`),
  KEY `idx_contract_info_appNum_carType` (`app_num`,`car_type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='合同基本信息表' ;
 
 
SELECT DISTINCT
        a.queue_id,
        c.app_num,
        c.province,
        c.city,
        b.nickname 用户ID,
        c.application_number 合同号,
        c.overdue_days 逾期天数,
        d.`name` 主借人,
        c.overdue_receivables 逾期金额,
        c.residual_amount 未偿本金,
        ( SELECT aa.act_sign FROM afp_record_new aa WHERE aa.afp_id = c.id ORDER BY input_time DESC LIMIT 1 ) 行动代码
FROM
        contract_info c
        LEFT JOIN mission_info a ON a.afp_id=c.id
        LEFT JOIN sys_user b on b.id= a.user_id
        LEFT JOIN customer_basic d on d.afp_id = c.id
WHERE
        d.role_name = '主借人'
        AND c.overdue_days BETWEEN 1 and 91
        AND a.queue_id IN (86, 87, 165, 115, 116, 117, 118, 119, 169, 170, 171, 172, 173 ,176)
ORDER BY
        nickname,
        overdue_days;
``` 

发现在cp_buffer_from_ref的过程中, s_key 中的内存被污染, 在相关内存上设置watchpoint

```
(gdb) p *ref
$126 = {key_err = true, has_record = false, key_parts = 1, key_length = 259, key = 1,
  key_buff = 0x7f4160ce50b0 "", key_buff2 = 0x7f4160ce51b8 "",
  key_copy = 0x7f4160ce52c0, items = 0x7f4160ce52c8, cond_guards = 0x7f4160ce52d0,
  null_rejecting = 0, depend_map = 0, null_ref_key = 0x0, use_count = 0,
  disable_cache = true}
(gdb) p *ref->key_copy
$127 = (store_key *) 0x7f4160ce52e8
(gdb) p *((store_key_field*) 0x7f4160ce52e8)
$128 = {<store_key> = {_vptr.store_key = 0x303f4d8 <vtable for store_key_field+16>,
    null_key = false, to_field = 0x7f4160ce5388, null_ptr = 0x7f4160ce50b0 "",
    err = 0 '\000'}, copy_field = {from_ptr = 0x7f4160c383bd "g.000035",
    to_ptr = 0x7f4160ce50b1 "", from_null_ptr = 0x0, to_null_ptr = 0x7f4160ce50b0 "",
    from_bit = 0, to_bit = 1, tmp = {m_ptr = 0x0, m_length = 0,
      m_charset = 0x31b6520 <my_charset_bin>, m_alloced_length = 0,
      m_is_alloced = false}, m_do_copy = 0xd9dd60 <do_copy_maybe_null(Copy_field*)>,
    m_do_copy2 = 0xd9da30 <do_varstring(Copy_field*)>, m_from_length = 258,
    m_to_length = 258, m_from_field = 0x7f4160c29730, m_to_field = 0x7f4160ce5388},
  field_name = 0x7f4160ce52d8 "x22.c.id"}
(gdb) p *((store_key_field*) 0x7f4160ce52e8)->to_field
$129 = {<Proto_field> = {
    _vptr.Proto_field = 0x3055ed0 <vtable for Field_varstring+16>},
  ptr = 0x7f4160ce50b1 "", m_null_ptr = 0x7f4160ce50b0 "", m_is_tmp_nullable = false,
  m_is_tmp_null = false, m_check_for_truncated_fields_saved = CHECK_FIELD_IGNORE,
  table = 0x7f4160ba1490, orig_table = 0x7f4160ba1490, table_name = 0x7f4160ba15a8,
  field_name = 0x7f4160c312b8 "afp_id", comment = {str = 0x7f4160c31370 "案件ID",
    length = 8}, key_start = {map = 0}, part_of_key = {map = 0}, part_of_prefixkey = {
    map = 0}, part_of_sortkey = {map = 0}, part_of_key_not_extended = {map = 2},
  field_length = 256, flags = 0, field_index = 1, null_bit = 1 '\001',
  auto_flags = 0 '\000', is_created_from_null_item = false, m_indexed = false,
  m_warnings_pushed = 0, gcol_info = 0x0, stored_in_db = true}
(gdb) p &(((store_key_field*) 0x7f4160ce52e8)->to_field)
$131 = (Field **) 0x7f4160ce52f8
(gdb) watch *0x7f4160ce52f8
Hardware watchpoint 16: *0x7f4160ce52f8
(gdb) c
Continuing.
``` 

发现内存在以下堆栈被修改: 

```
Thread 38 "mysqld" hit Hardware watchpoint 16: *0x7f4160ce52f8

Old value = 1624134536
New value = 1970167662
 
(gdb) bt
#0  0x00007f41e060d5cc in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x0000000000d9d94b in copy_field_varstring (to=0x7f4160ce5388, from=0x7f4160c29730)
    at ../../mysql-8.0.11/sql/field_conv.cc:549
#2  0x0000000000d9dd15 in Copy_field::invoke_do_copy2 (this=<optimized out>,
    f=0x7f4160ce5310) at ../../mysql-8.0.11/sql/field_conv.cc:568
#3  0x0000000000d9dcb5 in Copy_field::invoke_do_copy (this=this@entry=0x7f4160ce5310,
    f=f@entry=0x7f4160ce5310) at ../../mysql-8.0.11/sql/field_conv.cc:562
#4  0x0000000000c0c548 in store_key_field::copy_inner (this=0x7f4160ce52e8)
    at ../../mysql-8.0.11/sql/sql_select.h:980
#5  0x0000000000b977bc in store_key::copy (this=<optimized out>)
    at ../../mysql-8.0.11/sql/sql_select.h:919
#6  cp_buffer_from_ref (thd=thd@entry=0x7f4160000db0,
    table=table@entry=0x7f4160ba1490, ref=ref@entry=0x7f4160ce4308)
    at ../../mysql-8.0.11/sql/sql_executor.cc:5656
#7  0x0000000000b26b19 in get_quick_select_for_ref (thd=0x7f4160000db0,
    table=0x7f4160ba1490, ref=0x7f4160ce4308, records=1)
    at ../../mysql-8.0.11/sql/opt_range.cc:10248
#8  0x0000000000c13915 in JOIN::add_sorting_to_table (this=this@entry=0x7f4160ce3dc0,
    idx=idx@entry=0, sort_order=sort_order@entry=0x7f41cdcb6c80,
    force_stable_sort=force_stable_sort@entry=false)
    at ../../mysql-8.0.11/sql/sql_select.cc:4393
#9  0x0000000000c13c90 in JOIN::make_tmp_tables_info (this=this@entry=0x7f4160ce3dc0)
    at ../../mysql-8.0.11/sql/sql_select.cc:4136
#10 0x0000000000bc3a26 in JOIN::optimize (this=0x7f4160ce3dc0)
    at ../../mysql-8.0.11/sql/sql_optimizer.cc:722
#11 0x0000000000c0ee32 in SELECT_LEX::optimize (this=this@entry=0x7f4160c33598,
    thd=thd@entry=0x7f4160000db0) at ../../mysql-8.0.11/sql/sql_select.cc:1488
#12 0x0000000000c6d1ab in SELECT_LEX_UNIT::optimize (this=this@entry=0x7f4160c33890,
    thd=thd@entry=0x7f4160000db0) at ../../mysql-8.0.11/sql/sql_union.cc:752
#13 0x0000000000c0ee81 in SELECT_LEX::optimize (this=<optimized out>,
    thd=0x7f4160000db0) at ../../mysql-8.0.11/sql/sql_select.cc:1495
#14 0x0000000000c0f324 in Sql_cmd_dml::execute_inner (this=0x7f4160c0c8e8,
    thd=0x7f4160000db0) at ../../mysql-8.0.11/sql/sql_select.cc:640
#15 0x0000000000c17547 in Sql_cmd_dml::execute (this=0x7f4160c0c8e8,
    thd=0x7f4160000db0) at ../../mysql-8.0.11/sql/sql_select.cc:554
#16 0x0000000000bcbf2f in mysql_execute_command (thd=thd@entry=0x7f4160000db0,
    first_level=first_level@entry=true) at ../../mysql-8.0.11/sql/sql_parse.cc:4220
#17 0x0000000000bd040f in mysql_parse (thd=thd@entry=0x7f4160000db0,
    parser_state=parser_state@entry=0x7f41cdcb85e0)
    at ../../mysql-8.0.11/sql/sql_parse.cc:4935
#18 0x0000000000bd2d3a in dispatch_command (thd=thd@entry=0x7f4160000db0,
``` 

认为 varchar长度的计算可能有错误: 

![image2021-12-12 0:58:7.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-12%200%3A58%3A7.png)

调试得知: 

get_varstring_copy_length, 从from的前两字节获取长度, 而该from中得出的长度是 11879, (to->ptr + 11879) 从内存上刚好会污染到之前的s_key (copy_field)

![image2021-12-12 1:23:35.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-12%201%3A23%3A35.png)

追踪key_copy的来源堆栈: 

断点create_ref_for_key, 监听 p *j->m_qs->m_table

```
#0  create_ref_for_key (join=join@entry=0x7f1a28c6be40, j=0x7f1a28c6c2c0,
    org_keyuse=<optimized out>, used_tables=<optimized out>)
    at ../../mysql-8.0.11/sql/sql_select.cc:1804
#1  0x0000000000c16630 in JOIN::init_ref_access (this=this@entry=0x7f1a28c6be40)
    at ../../mysql-8.0.11/sql/sql_select.cc:1562
#2  0x0000000000bc22a7 in JOIN::optimize (this=0x7f1a28c6be40)
    at ../../mysql-8.0.11/sql/sql_optimizer.cc:502
#3  0x0000000000c0ee32 in SELECT_LEX::optimize (this=this@entry=0x7f1a28bb32d8,
    thd=thd@entry=0x7f1a28000db0) at ../../mysql-8.0.11/sql/sql_select.cc:1488
#4  0x0000000000c6d1ab in SELECT_LEX_UNIT::optimize (this=this@entry=0x7f1a28bb35d0,
    thd=thd@entry=0x7f1a28000db0) at ../../mysql-8.0.11/sql/sql_union.cc:752
#5  0x0000000000c0ee81 in SELECT_LEX::optimize (this=<optimized out>,
    thd=0x7f1a28000db0) at ../../mysql-8.0.11/sql/sql_select.cc:1495
#6  0x0000000000c0f324 in Sql_cmd_dml::execute_inner (this=0x7f1a28110518,
    thd=0x7f1a28000db0) at ../../mysql-8.0.11/sql/sql_select.cc:640
#7  0x0000000000c17547 in Sql_cmd_dml::execute (this=0x7f1a28110518,
    thd=0x7f1a28000db0) at ../../mysql-8.0.11/sql/sql_select.cc:554
#8  0x0000000000bcbf2f in mysql_execute_command (thd=thd@entry=0x7f1a28000db0,
    first_level=first_level@entry=true) at ../../mysql-8.0.11/sql/sql_parse.cc:4220
#9  0x0000000000bd040f in mysql_parse (thd=thd@entry=0x7f1a28000db0,
    parser_state=parser_state@entry=0x7f1a949205e0)
    at ../../mysql-8.0.11/sql/sql_parse.cc:4935
#10 0x0000000000bd2d3a in dispatch_command (thd=thd@entry=0x7f1a28000db0,
    com_data=com_data@entry=0x7f1a94920ce0, command=<optimized out>)
    at ../../mysql-8.0.11/sql/sql_parse.cc:1589
#11 0x0000000000bd3be9 in do_command (thd=thd@entry=0x7f1a28000db0)
    at ../../mysql-8.0.11/sql/sql_parse.cc:1214
#12 0x0000000000cbfd80 in handle_connection (arg=arg@entry=0x4cdeb20)
    at ../../mysql-8.0.11/sql/conn_handler/connection_handler_per_thread.cc:308
#13 0x0000000001aa5f5f in pfs_spawn_thread (arg=0x4c9ab30)
    at ../../../mysql-8.0.11/storage/perfschema/pfs.cc:2814
#14 0x00007f1aa8f996db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#15 0x00007f1aa72db71f in clone () from /lib/x86_64-linux-gnu/libc.so.6
``` 

断点设置在此处: 

![image2021-12-13 14:20:53.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-13%2014%3A20%3A53.png)

确认s_key在get_store_key后就受到污染: 

![image2021-12-14 23:13:36.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-14%2023%3A13%3A36.png)

而s_key.copy_field.from_ptr, 来自于keyuse:

![image2021-12-14 23:17:13.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-14%2023%3A17%3A13.png)

keyuse来自于 tab->position()->key, 断点设置在如下位置

![image2021-12-14 23:27:46.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-14%2023%3A27%3A46.png)

从best_ref中取出时, 看到内存已经污染: 

![image2021-12-14 23:28:40.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-14%2023%3A28%3A40.png)

找到构造[c.id](<http://c.id>)的Field的位置, 断点设置在 Item_field 构造器. 检查构造后, ptr是否污染, watch相关Item_field.field的变更位置

找到如下位置: 

```
(gdb) bt
#0  Item_field::set_field (this=this@entry=0x7fd624b80448, field_par=0x7fd624b8b1d0)
    at ../../mysql-8.0.11/sql/item.cc:2472
#1  0x0000000000df0503 in Item_field::fix_outer_field (this=this@entry=0x7fd624b80448,
    thd=thd@entry=0x7fd624000dd0, from_field=from_field@entry=0x7fd65b64fae8,
    reference=reference@entry=0x7fd624b80600) at ../../mysql-8.0.11/sql/item.cc:4794
#2  0x0000000000df1302 in Item_field::fix_fields (this=0x7fd624b80448,
    thd=0x7fd624000dd0, reference=0x7fd624b80600)
    at ../../mysql-8.0.11/sql/item.cc:5206
#3  0x0000000000e46169 in Item_func::fix_func_arg (this=this@entry=0x7fd624b80560,
    thd=thd@entry=0x7fd624000dd0, arg=arg@entry=0x7fd624b80600)
    at ../../mysql-8.0.11/sql/item_func.cc:292
#4  0x0000000000e4623b in Item_func::fix_fields (this=0x7fd624b80560,
    thd=0x7fd624000dd0) at ../../mysql-8.0.11/sql/item_func.cc:281
#5  0x0000000000c02f57 in SELECT_LEX::setup_conds (this=this@entry=0x7fd624b7f318,
    thd=thd@entry=0x7fd624000dd0) at ../../mysql-8.0.11/sql/sql_resolver.cc:1263
#6  0x0000000000c09674 in SELECT_LEX::prepare (this=0x7fd624b7f318,
    thd=thd@entry=0x7fd624000dd0) at ../../mysql-8.0.11/sql/sql_resolver.cc:264
#7  0x0000000000e824c4 in subselect_single_select_engine::prepare (this=0x7fd624b80920)
    at ../../mysql-8.0.11/sql/item_subselect.cc:2775
#8  0x0000000000e89b9d in Item_subselect::fix_fields (this=0x7fd624b80808,
    thd=0x7fd624000dd0, ref=0x7fd624169a38)
    at ../../mysql-8.0.11/sql/item_subselect.cc:576
#9  0x0000000000b6c881 in setup_fields (thd=thd@entry=0x7fd624000dd0,
    ref_item_array=..., fields=..., want_privilege=1,
    sum_func_list=sum_func_list@entry=0x7fd62400d948, allow_sum_func=true,
    column_update=false) at ../../mysql-8.0.11/sql/sql_base.cc:8535
#10 0x0000000000c095ff in SELECT_LEX::prepare (this=this@entry=0x7fd62400d7d8,
    thd=thd@entry=0x7fd624000dd0) at ../../mysql-8.0.11/sql/sql_resolver.cc:248
#11 0x0000000000c0e035 in Sql_cmd_select::prepare_inner (this=0x7fd624176c78,
    thd=0x7fd624000dd0) at ../../mysql-8.0.11/sql/sql_select.cc:427
#12 0x0000000000c0c232 in Sql_cmd_dml::prepare (this=0x7fd624176c78,
    thd=0x7fd624000dd0) at ../../mysql-8.0.11/sql/sql_select.cc:366
#13 0x0000000000c174ba in Sql_cmd_dml::execute (this=0x7fd624176c78,
    thd=0x7fd624000dd0) at ../../mysql-8.0.11/sql/sql_select.cc:494
#14 0x0000000000bcbf2f in mysql_execute_command (thd=thd@entry=0x7fd624000dd0,
    first_level=first_level@entry=true) at ../../mysql-8.0.11/sql/sql_parse.cc:4220
#15 0x0000000000bd040f in mysql_parse (thd=thd@entry=0x7fd624000dd0,
    parser_state=parser_state@entry=0x7fd65b6515e0)
    at ../../mysql-8.0.11/sql/sql_parse.cc:4935
#16 0x0000000000bd2d3a in dispatch_command (thd=thd@entry=0x7fd624000dd0,
    com_data=com_data@entry=0x7fd65b651ce0, command=<optimized out>)
``` 

field已被污染

![image2021-12-15 7:0:5.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-15%207%3A0%3A5.png)

找到field的来源: 

```
(gdb) bt
#0  find_field_in_table (table=0x7f32e8190150, name=name@entry=0x7f32e8122588 "id",
    length=length@entry=2, allow_rowid=<optimized out>,
    cached_field_index_ptr=cached_field_index_ptr@entry=0x7f32e80eb3b0)
    at ../../mysql-8.0.11/sql/sql_base.cc:7163
#1  0x0000000000b6ee4d in find_field_in_table_ref (thd=thd@entry=0x7f32e8000db0,
    table_list=table_list@entry=0x7f32e80eb800, name=name@entry=0x7f32e8122588 "id",
    length=length@entry=2, item_name=0x7f32e8122588 "id", db_name=db_name@entry=0x0,
    table_name=0x7f32e8122578 "c", ref=0x7f32e80eb4a0, want_privilege=1,
    allow_rowid=true, cached_field_index_ptr=0x7f32e80eb3b0,
    register_tree_change=true, actual_table=0x7f3348ec8798)
    at ../../mysql-8.0.11/sql/sql_base.cc:7294
#2  0x0000000000b6f29e in find_field_in_tables (thd=thd@entry=0x7f32e8000db0,
    item=item@entry=0x7f32e80eb2e8, first_table=0x7f32e80eb800, last_table=0x0,
    ref=ref@entry=0x7f32e80eb4a0,
    report_error=report_error@entry=IGNORE_EXCEPT_NON_UNIQUE, want_privilege=1,
    register_tree_change=true) at ../../mysql-8.0.11/sql/sql_base.cc:7514
#3  0x0000000000df0749 in Item_field::fix_outer_field (this=this@entry=0x7f32e80eb2e8,
    thd=thd@entry=0x7f32e8000db0, from_field=from_field@entry=0x7f3348ec8ae8,
    reference=reference@entry=0x7f32e80eb4a0) at ../../mysql-8.0.11/sql/item.cc:4785
#4  0x0000000000df1302 in Item_field::fix_fields (this=0x7f32e80eb2e8,
    thd=0x7f32e8000db0, reference=0x7f32e80eb4a0)
    at ../../mysql-8.0.11/sql/item.cc:5206
#5  0x0000000000e46169 in Item_func::fix_func_arg (this=this@entry=0x7f32e80eb400,
    thd=thd@entry=0x7f32e8000db0, arg=arg@entry=0x7f32e80eb4a0)
    at ../../mysql-8.0.11/sql/item_func.cc:292
#6  0x0000000000e4623b in Item_func::fix_fields (this=0x7f32e80eb400,
    thd=0x7f32e8000db0) at ../../mysql-8.0.11/sql/item_func.cc:281
#7  0x0000000000c02f57 in SELECT_LEX::setup_conds (this=this@entry=0x7f32e80ea1b8,
    thd=thd@entry=0x7f32e8000db0) at ../../mysql-8.0.11/sql/sql_resolver.cc:1263
#8  0x0000000000c09674 in SELECT_LEX::prepare (this=0x7f32e80ea1b8,
    thd=thd@entry=0x7f32e8000db0) at ../../mysql-8.0.11/sql/sql_resolver.cc:264
#9  0x0000000000e824c4 in subselect_single_select_engine::prepare (this=0x7f32e80eb7c0)
    at ../../mysql-8.0.11/sql/item_subselect.cc:2775
#10 0x0000000000e89b9d in Item_subselect::fix_fields (this=0x7f32e80eb6a8,
    thd=0x7f32e8000db0, ref=0x7f32e8122bf8)
    at ../../mysql-8.0.11/sql/item_subselect.cc:576
#11 0x0000000000b6c881 in setup_fields (thd=thd@entry=0x7f32e8000db0,
    ref_item_array=..., fields=..., want_privilege=1,
    sum_func_list=sum_func_list@entry=0x7f32e800f958, allow_sum_func=true,
    column_update=false) at ../../mysql-8.0.11/sql/sql_base.cc:8535
#12 0x0000000000c095ff in SELECT_LEX::prepare (this=this@entry=0x7f32e800f7e8,
--Type <RET> for more, q to quit, c to continue without paging--
    thd=thd@entry=0x7f32e8000db0) at ../../mysql-8.0.11/sql/sql_resolver.cc:248
#13 0x0000000000c0e035 in Sql_cmd_select::prepare_inner (this=0x7f32e8173008,
    thd=0x7f32e8000db0) at ../../mysql-8.0.11/sql/sql_select.cc:427
#14 0x0000000000c0c232 in Sql_cmd_dml::prepare (this=0x7f32e8173008,
    thd=0x7f32e8000db0) at ../../mysql-8.0.11/sql/sql_select.cc:366
#15 0x0000000000c174ba in Sql_cmd_dml::execute (this=0x7f32e8173008,
    thd=0x7f32e8000db0) at ../../mysql-8.0.11/sql/sql_select.cc:494
#16 0x0000000000bcbf2f in mysql_execute_command (thd=thd@entry=0x7f32e8000db0,
    first_level=first_level@entry=true) at ../../mysql-8.0.11/sql/sql_parse.cc:4220
#17 0x0000000000bd040f in mysql_parse (thd=thd@entry=0x7f32e8000db0,
    parser_state=parser_state@entry=0x7f3348eca5e0)
    at ../../mysql-8.0.11/sql/sql_parse.cc:4935
#18 0x0000000000bd2d3a in dispatch_command (thd=thd@entry=0x7f32e8000db0,
    com_data=com_data@entry=0x7f3348ecace0, command=<optimized out>)
    at ../../mysql-8.0.11/sql/sql_parse.cc:1589
#19 0x0000000000bd3be9 in do_command (thd=thd@entry=0x7f32e8000db0)
    at ../../mysql-8.0.11/sql/sql_parse.cc:1214
#20 0x0000000000cbfd80 in handle_connection (arg=arg@entry=0x4c44160)
    at ../../mysql-8.0.11/sql/conn_handler/connection_handler_per_thread.cc:308
#21 0x0000000001aa5f5f in pfs_spawn_thread (arg=0x4c52170)
    at ../../../mysql-8.0.11/storage/perfschema/pfs.cc:2814
#22 0x00007f335d5436db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#23 0x00007f335b88571f in clone () from /lib/x86_64-linux-gnu/libc.so.6
``` 

是从TABLE.field中获取:

![image2021-12-15 7:43:26.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-15%207%3A43%3A26.png)

找到TABLE的构造处 open_table_from_share, 找到field的来源: 

```
(gdb) bt
#0  open_table_from_share (thd=thd@entry=0x7fe0ac000db0,
    share=share@entry=0x7fe0ac0ea9b8, alias=<optimized out>, db_stat=db_stat@entry=39,
    prgflag=prgflag@entry=8, ha_open_flags=0, outparam=0x7fe0accc4190,
    is_create_table=false, table_def_param=0x7fe0acb76550)
    at ../../mysql-8.0.11/sql/table.cc:2804
#1  0x0000000000b740f4 in open_table (thd=thd@entry=0x7fe0ac000db0,
    table_list=table_list@entry=0x7fe0ac173760, ot_ctx=ot_ctx@entry=0x7fe114281060)
    at ../../mysql-8.0.11/sql/sql_base.cc:3296
#2  0x0000000000b7967c in open_and_process_table (ot_ctx=0x7fe114281060,
    has_prelocking_list=false, prelocking_strategy=0x7fe1142810f8,
    counter=0x7fe0ac003a38, tables=0x7fe0ac173760, lex=<optimized out>,
    thd=0x7fe0ac000db0) at ../../mysql-8.0.11/sql/sql_base.cc:4946
#3  open_tables (thd=thd@entry=0x7fe0ac000db0, start=start@entry=0x7fe1142810e8,
    counter=<optimized out>, flags=flags@entry=0,
    prelocking_strategy=prelocking_strategy@entry=0x7fe1142810f8)
    at ../../mysql-8.0.11/sql/sql_base.cc:5571
#4  0x0000000000b7a178 in open_tables_for_query (thd=thd@entry=0x7fe0ac000db0,
    tables=<optimized out>, flags=0) at ../../mysql-8.0.11/sql/sql_base.cc:6351
#5  0x0000000000c0c1e1 in Sql_cmd_dml::prepare (this=0x7fe0acc77258,
    thd=0x7fe0ac000db0) at ../../mysql-8.0.11/sql/sql_select.cc:345
#6  0x0000000000c174ba in Sql_cmd_dml::execute (this=0x7fe0acc77258,
    thd=0x7fe0ac000db0) at ../../mysql-8.0.11/sql/sql_select.cc:494
#7  0x0000000000bcbf2f in mysql_execute_command (thd=thd@entry=0x7fe0ac000db0,
    first_level=first_level@entry=true) at ../../mysql-8.0.11/sql/sql_parse.cc:4220
#8  0x0000000000bd040f in mysql_parse (thd=thd@entry=0x7fe0ac000db0,
    parser_state=parser_state@entry=0x7fe1142825e0)
    at ../../mysql-8.0.11/sql/sql_parse.cc:4935
#9  0x0000000000bd2d3a in dispatch_command (thd=thd@entry=0x7fe0ac000db0,
    com_data=com_data@entry=0x7fe114282ce0, command=<optimized out>)
    at ../../mysql-8.0.11/sql/sql_parse.cc:1589
#10 0x0000000000bd3be9 in do_command (thd=thd@entry=0x7fe0ac000db0)
    at ../../mysql-8.0.11/sql/sql_parse.cc:1214
#11 0x0000000000cbfd80 in handle_connection (arg=arg@entry=0x436e370)
    at ../../mysql-8.0.11/sql/conn_handler/connection_handler_per_thread.cc:308
#12 0x0000000001aa5f5f in pfs_spawn_thread (arg=0x437c660)
    at ../../../mysql-8.0.11/storage/perfschema/pfs.cc:2814
#13 0x00007fe1280fa6db in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
#14 0x00007fe12643c71f in clone () from /lib/x86_64-linux-gnu/libc.so.6
``` 

![image2021-12-15 8:1:59.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-15%208%3A1%3A59.png)

field会指向 record[0]

TABLE.record的初始化位置为: 

![image2021-12-15 8:4:14.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-15%208%3A4%3A14.png)

# 结论1:

找到open_table_from_share中field的处理位置, 最新代码: 

![image2021-12-15 8:14:36.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-15%208%3A14%3A36.png)

找到field->reset的出处: 

<https://github.com/mysql/mysql-server/commit/5cb06a065470219747cf8d325e3be836bb8cd33e>

最低修复版本: 8.0.17

# 结论2:

alloc_root会分配未初始化的内存块, 最新代码中使用ArrayAlloc分配内存, 其中会进行内存的初始化: 

![image2021-12-15 8:35:0.png](/assets/01KJBYFF2KJAHSJFDW23BMQCHV/image2021-12-15%208%3A35%3A0.png)

相关commit:

<https://github.com/mysql/mysql-server/commit/7879ddb9aef3480afba391c76356ba38d566c957>

最低修复版本: 8.0.17

# 验证

使用8.0.12 - 8.0.16验证, 无法重现问题, 还有未知原因

方法1:

查看8.0.12的release note, 找到可能相关的commit, 逐个apply到8.0.11, 进行测试, 可以定位相关的commit

git命令: 

```
git format-patch -1 38ed24ee0631507c24ff77d2f09e57748d08a040
mv 0001-Bug-12635103-VALGRIND-WARNINGS-IN-MY_WILDCMP_MB-AND-.patch /tmp/
git am < /tmp/0001-Bug-12635103-VALGRIND-WARNINGS-IN-MY_WILDCMP_MB-AND-.patch
``` 

方法2:

git log --reverse --pretty=oneline mysql-8.0.11..mysql-8.0.12

获取commit列表

二分法确定相关commit:

<https://github.com/mysql/mysql-server/commit/f5017c9a6fd197372dc5c6ffb86d422d26183121>

再用方法1确认

# 最终结论

确认与 

<https://github.com/mysql/mysql-server/commit/f5017c9a6fd197372dc5c6ffb86d422d26183121>

相关
