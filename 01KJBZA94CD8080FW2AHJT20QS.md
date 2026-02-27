---
title: 20240618 - ChatDBA: 对问题信息的抽取太过缩略
confluence_page_id: 2949754
created_at: 2024-06-18T16:17:41+00:00
updated_at: 2024-06-18T16:17:41+00:00
---

# 测试场景

```
MySQL在执行select语句的时候crash，应该如何排查？

---
MySQL的error log:
06:43:38 UTC - mysqld got signal 8 ;
This could be because you hit a bug. It is also possible that this binary
or one of the libraries it was linked against is corrupt, improperly built,
or misconfigured. This error can also be caused by malfunctioning hardware.
Attempting to collect some information that could help diagnose the problem.
As this is a crash and something is definitely wrong, the information
collection process might fail.key_buffer_size=8388608
read_buffer_size=2097152
max_used_connections=88
max_threads=1000
thread_count=64
connection_count=63
It is possible that mysqld could use up to 
key_buffer_size + (read_buffer_size + sort_buffer_size)*max_threads = 6165582 K  bytes of memory
Hope that's ok; if not, decrease some variables in the equation.Thread pointer: 0x7f3b0c2606c0
Attempting backtrace. You can use the following information to find out
where mysqld died. If you see no messages after this, something went
terribly wrong...
stack_bottom = 7f425356ce98 thread_stack 0x40000
/mysql/mysql57/bin/mysqld(my_print_stacktrace+0x35)[0xf7b545]
/mysql/mysql57/bin/mysqld(handle_fatal_signal+0x4b9)[0x7fab89]
/lib64/libpthread.so.0(+0x134c0)[0x7f48e59c24c0]
/mysql/mysql57/bin/mysqld(decimal2bin+0x1c0)[0x1487e90]
/mysql/mysql57/bin/mysqld(_Z17my_decimal2binaryjPK10my_decimalPhii+0xc5)[0xc916a5]
/mysql/mysql57/bin/mysqld(_ZN10Sort_param12make_sortkeyEPhPKh+0x33d)[0x83bf7d]
/mysql/mysql57/bin/mysqld(_Z8filesortP3THDP8FilesortbPyS3_S3_+0x117b)[0x83f6eb]
/mysql/mysql57/bin/mysqld(_ZN7QEP_TAB10sort_tableEv+0x10f)[0xd0f49f]
/mysql/mysql57/bin/mysqld(_Z21join_init_read_recordP7QEP_TAB+0x2e)[0xd0f60e]
/mysql/mysql57/bin/mysqld(_Z10sub_selectP4JOINP7QEP_TABb+0x2ba)[0xd1115a]
/mysql/mysql57/bin/mysqld(_ZN4JOIN4execEv+0x27a)[0xd1068a]
/mysql/mysql57/bin/mysqld(_Z12handle_queryP3THDP3LEXP12Query_resultyy+0x250)[0xd7bce0]
/mysql/mysql57/bin/mysqld[0xd3c913]
/mysql/mysql57/bin/mysqld(_Z21mysql_execute_commandP3THDb+0x385a)[0xd4044a]
/mysql/mysql57/bin/mysqld(_Z11mysql_parseP3THDP12Parser_state+0x3cd)[0xd41fcd]
/mysql/mysql57/bin/mysqld(_Z16dispatch_commandP3THDPK8COM_DATA19enum_server_command+0x1780)[0xd437f0]
/mysql/mysql57/bin/mysqld(_Z10do_commandP3THD+0x194)[0xd443b4]
/mysql/mysql57/bin/mysqld(handle_connection+0x2ac)[0xe1787c]
/mysql/mysql57/bin/mysqld(pfs_spawn_thread+0x174)[0x13e5bc4]
/lib64/libpthread.so.0(+0x8f1b)[0x7f48e59b7f1b]
/lib64/libc.so.6(clone+0x40)[0x7f48e556a340]Trying to get some variables.
Some pointers may be invalid and cause the dump to abort.
Query (7f3b0c13b880): select a.bank_org_longitude bankOrgLongitude,a.bank_org_latitude bankOrgLatitude,a.visit_range visitRange,  b.id id,b.org_name orgName ,b.address address,b.cust_org_code custOrgCode         from biz_bank_org_visit_rule a join biz_organization b on a.bank_org_id = b.id where bank_org_longitude is not null and bank_org_latitude is not null         and bank_org_id in  (                     2056933057096581120            )         ORDER BY ABS(0.000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000005-`bank_org_longitude`)+ABS(0.000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
Connection ID (thread ID): 276058
Status: NOT_KILLEDThe manual page at http://dev.mysql.com/doc/mysql/en/crashing.html contains
information that should help you find out what is causing the crash.
``` 

第二轮对话的完整日志: [prompt.txt](/assets/01KJBZA94CD8080FW2AHJT20QS/prompt.txt)

# 出现的问题

在答案中, 提到了DuplicateWeedout, 但实际上与DuplicateWeedout无关: 

````
	分析表明，问题与MySQL 5.7版本中的Decimal类型处理和'DuplicateWeedout'优化策略有关，这可能导致内存或优化器错误。'decimal2bin'函数的使用可能触发了这个问题。以下是解决建议：
	
	1. **升级MySQL**: 最佳解决方案是将MySQL升级到8.0.24或更高版本，以修复已知的优化器错误。在升级前，请确保做好数据备份和兼容性检查。
	
	2. **临时关闭DuplicateWeedout策略**：
	   - 如果无法立即升级，可以尝试在当前会话中临时关闭DuplicateWeedout：
	     ```sql
	     SET optimizer_switch='duplicateweedout=off';
	     ```
	   - 或者在`my.cnf`配置文件中永久关闭，然后重启MySQL服务：
	     ```conf
	     optimizer_switch='duplicateweedout=off'
	     ```
	
	为了确定最佳解决方案，请提供以下信息：
	- 查询涉及的表结构，尤其是`bank_org_longitude`列的数据类型和Decimal列的具体信息。
	- 是否有其他查询或表出现类似错误。
	- 业务需求紧迫性以及服务器硬件是否支持升级MySQL。
	
	这些信息将帮助我们评估风险并提供最合适的解决方案。在实施任何更改之前，请务必评估对业务的影响。
```` 

分析Plan: 

```
SWITCH('MySQL执行select语句crash')
Case{
    THINKING(可能是由于内存不足或优化器错误导致的崩溃)
    FIND(查看错误日志)
    IF (错误日志中有 'mysqld got signal 11'):
        THINKING(可能是内存溢出导致的崩溃)
        FIND(检查当前内存使用情况和配置)
        IF (内存使用接近系统限制或内存配置较低):
            REASON_CONFIRMED(内存资源不足)
            RETURN(增加内存配置或优化内存使用)
    Case{
        THINKING(可能是由于特定优化策略导致的崩溃)
        FIND(分析crash堆栈信息)
        IF (堆栈信息表明是由于优化器错误引起):
            THINKING(可能是DuplicateWeedout优化策略的问题)
            FIND(查看MySQL版本)
            IF (MySQL版本低于8.0.24):
                REASON_CONFIRMED(存在已知的优化器错误)
                RETURN(升级MySQL到8.0.24或关闭DuplicateWeedout策略)
            ELSE:
                THINKING(可能是其他优化器问题, 需要深入代码分析)
                RETURN(进一步分析代码和官方文档以确定原因)
    }
    Case{
        THINKING(可能是由于数据类型不兼容或字符集问题)
        FIND(检查涉及的表和字段的数据类型和字符集)
        IF (存在不兼容的数据类型或字符集):
            REASON_CONFIRMED(数据类型或字符集不兼容)
            RETURN(调整数据类型或统一字符集和排序规则)
    }
}
``` 
    
    
     

  
问题

  1. "IF (堆栈信息表明是由于优化器错误引起)" 这一步的判断指标太过简略, 大模型不一定有足够的知识
  2. "REASON_CONFIRMED(存在已知的优化器错误)" 这一步, 应当指明具体的错误链接, 或者提供一些背景知识

为什么Plan生成得如此简单? 在生成Plan时, 使用的参考文档: 

```
<参考文档>
[问题总结]
问题总结: MySQL执行select语句导致数据库crash, error日志报错mysqld got signal 11

[现象描述]
现象描述:
- 数据库执行一条select语句就会导致异常crash
- MySQL error日志报错mysqld got signal 11
- MySQL版本：Ver 8.0.19 for linux-glibc2.12 on x86_64(MySQL Community Server - GPL)

[问题原因]
问题原因:
- 优化器将EXISTS子查询转换为semi-join操作，采用DuplicateWeedout执行策略，导致建立临时表时内存地址错乱

[解决方法]
问题解决方法:
1. 升级MySQL版本到8.0.24
2. 设置业务表统一的字符集和排序规则
3. 关闭数据库级别的DuplicateWeedout优化策略：SET[GLOBAL|SESSION] optimizer_switch='duplicateweedout=off'

[排查步骤]
问题排查步骤:
  1. 查看监控信息，排查数据库崩溃时是否有异常输出。
  2. 分析 crash 堆栈信息，确定异常是由 join 查询操作引起。
  3. 通过执行计划分析，验证优化器将 EXISTS 子查询转换为 semi-join 操作，并采用 DuplicateWeedout 执行策略。
  4. 使用 gdb 定位到崩溃代码位置，发现 actual_key_parts 函数中 in_use 返回值为空，说明 table 内存取址错误。
  5. 确定该 bug 是由于使用了 semi-join 的 DuplicateWeedout 执行策略导致的。
  6. 验证在 MySQL 8.0.24 版本后对该代码进行了修复。
  7. 提出解决建议：
    - 避免执行触发崩溃的 SQL。
    - 设置统一的字符集和排序规则，避免使用 DuplicateWeedout 策略。
    - 关闭 DuplicateWeedout 优化策略。
    - 升级 MySQL 版本到 8.0.24。
  8. 提供其他解决思路：
    - 检索类似 bug 并验证修复版本。
    - 开启 coredump 完成堆栈辅助验证。

[指令块]
命令1：[`SELECT DISTINCT T.CUST_NO FROM AML_DM.TA00_TRANSACTION T  WHERE EXISTS (SELECT 1  FROM AML_DM.tb53_new_cust_i_newtest_03 T1 WHERE T.CUST_NO = T1.CUST_NO ) AND T.AGENT_CERT_NO IS NOT NULL`]
命令1的解释：[该命令用于从 AML_DM.TA00_TRANSACTION 表中查询出符合条件的客户号 (CUST_NO)。条件是：1. 该客户号存在于 AML_DM.tb53_new_cust_i_newtest_03 表中；2. 该客户的代理人证书编号 (AGENT_CERT_NO) 不为空。]

命令2：[`SELECT 'PJC1020' AS INDIC_KEY, A.CUST_NO AS OBJ_KEY, CASE WHEN B.CUST_NO IS NULL THEN 1 ELSE END AS INDICVAL1,'2022-06-06' AS GRADING_DATE FROM AML_DM.TB53_NEW_CUST_I A LEFT JOIN ( SELECT DISTINCT T.CUST_NO FROM AML_DM.TA00_TRANSACTION T WHERE EXISTS ( SELECT 1 FROM AML_DM.TB53_NEW_CUST_I T1 WHERE T.CUST_NO = T1.CUST_NO ) AND T.AGENT_CERT_NO IS NOT NULL ) B ON A.CUST_NO = B.CUST_NO;`]
命令2的解释：[该命令使用左连接查询 AML_DM.TB53_NEW_CUST_I 和 AML_DM.TA00_TRANSACTION 两个表，并返回拼接后的结果。]

命令3：[`CREATE TABLE `ta00_transaction` (`cert_key` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '客户身份信息表对应主键', `cust_no` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '客户号', ... ]
命令3的解释：[该命令用于创建一个名为 ta00_transaction 的表，并定义表的列名、数据类型、默认值和注释。]

命令4：[`CREATE TABLE `tb53_new_cust_i` (`CUST_NO` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '客户号', ... ]
命令4的解释：[该命令用于创建一个名为 tb53_new_cust_i 的表，并定义表的列名、数据类型、默认值和注释。]
</参考文档>
``` 

其中并没有 如何判断堆栈的匹配度, 或者堆栈的具体信息, 只有简化后的逻辑. 

根据简化后的(有缺失信息的)逻辑, 大模型根据逻辑生成的Plan和后续的判断 逻辑上是正确的, 但缺失的信息导致判断结果错误. 

# 核心原因

简化文档时, 丢失了用于匹配问题的重要信息.
