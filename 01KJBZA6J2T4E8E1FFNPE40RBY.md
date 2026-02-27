---
title: 20240618 - ChatDBA: 如何判断参考材料和问题的相关性 [2]
confluence_page_id: 2949747
created_at: 2024-06-18T13:12:51+00:00
updated_at: 2024-06-18T15:48:45+00:00
---

# 背景

研究问答: "MySQL在执行select语句的时候crash，应该如何排查？"

完整日志: [q.txt](/assets/01KJBZA6J2T4E8E1FFNPE40RBY/q.txt)

# 问题

答案中会提到: "optimizer_switch"

```
[思考: 用户没有提供具体的信息, 但我看到了之前在错误日志中检查内存溢出或信号11的要求]
您在查看MySQL错误日志时，有发现提到内存溢出或者信号11吗？这类线索通常表明MySQL在处理 SELECT 语句时遇到了内存相关的问题。

根据您的反馈，我们会决定下一步的操作：
- 如果日志中提到了内存问题，那么我们倾向于相信内存限制可能是个关键因素。
- 如果没有提到这类问题，我们就需要考察其他可能导致crash的原因，例如优化器策略或数据类型的兼容性。

[思考: 下一阶段我们需要采集更多关于错误信息的信息, 以及其他几个重要变量来进一步确诊]
我们需要：
1. 提供查看到的具体错误日志内容，这可能包含崩溃时的具体错误信息。
2. `show processlist` 的结果，这样我们了解SELECT语句的内容。
3. MySQL的`optimizer_switch`变量设置的现状。
4. 如果方便的话，崩溃时涉及表的结构信息，尤其是数据类型和字符集。

这些都会帮助我们进一步缩小可能的原因，并找到最佳解决方案。
``` 

这部分和crash的直接关系不大, 需要经过较长的间接推导.

尝试研究其文档材料 与 问题的相关性

# 过程

生成空文档的执行计划

```
 llm = DashscopeGenerator(stage="llm_is_need_retrieve_document", api_key=DASHSCOPE_KEY, debugger=debugger)
prompt = """
你是一个DBA专家AI, 负责针对人类提出的MySQL数据库问题进行排查计划的生成.
下面是数据库服务工单, <数据库服务工单> 中的数据库问题可能与 <用户输入的MySQL数据库问题> 类似或一致, 所以你需要参考 <数据库服务工单> 中的内容来生成解决人类提出问题的排查计划.
排查计划的每个步骤必须能在 <数据库服务工单> 中可以直接获取, 或可以间接推理得到, 不能自己随意编造. 

排查计划不应该出现 <数据库服务工单> 中任何 '具体的值' 和 '具体的表名','字段名', 因为那是 <数据库服务工单> 中的背景信息并不是本次诊断的信息！！
排查计划不应该出现 <数据库服务工单> 中任何 '具体的值' 和 '具体的表名','字段名', 因为那是 <数据库服务工单> 中的背景信息并不是本次诊断的信息！！
排查计划不应该出现 <数据库服务工单> 中任何 '具体的值' 和 '具体的表名','字段名', 因为那是 <数据库服务工单> 中的背景信息并不是本次诊断的信息！！

AI需要学习以下 <数据库服务工单> 中的材料, 来理解分析 问题产生的 排查思路:
<数据库服务工单>
<数据库服务工单>

人类的最新输入, 是以下 <人类输入> 中的内容:
<人类输入>

MySQL在执行select语句的时候crash，应该如何排查？

</人类输入>

排查评估计划还需要遵循如下要求: 
1. 排查评估计划 用 Python 伪代码 进行表示, 答案除了 Python 伪代码 不要包含其他内容. 否则答案将会出错.
2. 对排查评估计划的伪代码函数的要求如下:
    - SWITCH('问题现象')是一个开始函数, 表示初始的问题现象
    - Case{}是一个分支函数,括号内的所有节点内容都表示针对某一个可能原因的排查方法。所以你首先需要根据数据库服务工单梳理问题可能原因有哪些，再将不同可能原因的排查方法使用Case函数分割开。
    - THINKING('在当前步骤的判断, 和下一步诊断的想法')是一个函数, 表示在当前位置, 我们能做出什么判断, 以及下一步诊断应当如何开展的想法，为FIND提供一个收集信息的原因
    - FIND(需要的信息)是一个函数, 这个函数的意思是 需要我提供指定信息, 对信息的描述应尽量具体.
    - IF(条件判断)是一个函数, 表示判别条件. 条件写得可以更具体一些, 让一个初级DBA能看懂. IF函数的条件必须是一个中文的判断句, 不能使用英文句子或纯英文代码. 
    - REASON_CONFIRMED(问题的根因)是一个函数, 表示执行到代码此处时, 已经确认问题的根因. 接下来的步骤 应该评估哪种解决方案更合适.
    - RETURN(xxx) 是一个结束排查的返回函数, 它表示执行到代码此处时, 给出问题的解决方案.
3. 对排查评估计划的伪代码的要求如下: 
    - 对于每一个FIND/IF/RETURN函数, 都需要给出一段函数实现, 给出具体的方法. 注意: 这个函数实现的语言不一定是Python, 遵从 <数据库服务工单>中的原始描述即可
    - 你只能使用 THINKING/FIND/IF/REASON_CONFIRMED/RETURN 函数, 不要使用其他函数, 不要使用"ELSE"函数
    - 你需要重点学习 THINKING/FIND/IF/REASON_CONFIRMED/RETURN 之间上下文 '逻辑关系', 保证你生成的答案逻辑是正确的
    - 在REASON_CONFIRMED之前, 是对问题的诊断流程; 在REASON_CONFIRMED之后, 是对解决方案的评估流程
    - 对于每一个FIND/IF/RETURN函数, 都需要给出一段函数实现, 给出具体的方法
    - 每一个Case函数中不能包含多个可能原因(REASON_CONFIRMED)
4. <数据库服务工单> 是虚构的场景和解决思路, 不是本次的真实场景, 不要使用参考资料中的信息作为你的思考依据和输出内容!!!
    - 不能直接引用 <数据库服务工单> 中出现的表名、字段名、文件名；不能将这些内容，视为已知内容继续进行排查评估计划的下一步
    - 不能将 <数据库服务工单> 中提供的描述：如提供的文件、描述的问题现象等内容，视为当前的问题；不能进行直接引用
5. 对排查评估计划的伪代码的逻辑复杂度 要求如下: 
    - 你需要仔细学习以下的 排查评估计划: 多级嵌套的逻辑 并不受欢迎, 应尽量让逻辑的嵌套层次变少
    - 你需要先对各个有用的信息逐一进行FIND, 在FIND之后用尽量少的步骤 确定 问题根因, 并给出解决方案
6. 对于IF(...)函数, 你需要严格遵守以下要求:
    - IF函数的参数必须是一个中文的判断句, 不能使用英文句子或纯英文代码. 
    - IF函数的参数, 禁止使用"过高/过低/过大/过小"这样的不可量化的判断
7. 为了保证诊断计划的可读性，你需要尽可能的保证整个计划的简洁，在不影响诊断逻辑的情况下，简化你的表述。
8. 注意: 将"排查评估计划"和"函数实现"分开列出, 不要将两部分信息都合并到 "排查评估计划" 中
9. 下面的 <排查评估计输出样例> 只可以参考它的输出结构, 不可以参考其中的内容, 否则会出错.

<排查评估计输出样例>
<样例1>
Switch(SQL运行慢)
Case{
IF (慢查询日志中 存在慢查询SQL):
    THINKING(SQL运行慢 可能跟锁等待有关, 也可能跟索引缺失有关)
    IF (慢查询SQL 锁等待时间较长):
        REASON_CONFIRMED(锁等待参数 可能小于 慢查询SQL的锁等待时间, 导致SQL锁超时)
        FIND(innodb_lock_wait_timeout参数设置)
        IF (innodb_lock_wait_timeout 小于 慢查询SQL的锁等待时间):
            THINKING(为了让SQL能执行完成, 需要调整锁超时的时间)
            RETURN(调整innodb_lock_wait_timeout参数设置)
}
Case{
THINKING(慢查询SQL 锁等待时间正常)
FIND(慢查询SQL相关的 表结构)
IF (表结构中 缺少相关索引):
    THINKING(缺少了跟SQL对应的索引, 会导致SQL运行慢, 可以添加索引解决)
    REASON_CONFIRMED(由于缺少索引, 导致SQL运行慢)
    RETURN(添加相关索引)
}
Case{
THINKING(SQL运行慢的问题并不是因为索引缺失引起的, 可尝试对SQL进行改写来优化)
REASON_CONFIRMED(SQL慢并不是因为参数或索引的原因, 需要尝试对SQL进行改写)
RETURN(优化SQL语句)
}

函数实现: 
FIND(慢查询日志)的方法:
    通过`SHOW GLOBAL VARIABLES LIKE 'slow_query_log'` 查看是否开启了慢日志
    通过`SHOW VARIABLES LIKE 'slow_query_log_file';` 查看慢日志位置
    如果已经开启了慢日志, 可以通过less对慢日志文件进行查看

RETURN(调整innodb_lock_wait_timeout参数设置)的方法:
    1. 查看当前配置： SHOW GLOBAL VARIABLES LIKE 'innodb_lock_wait_timeout';
    2. 修改当前配置（动态生效）： SET GLOBAL innodb_lock_wait_timeout=[new_value];

RETURN(添加索引)的方法:
    1. alter table table_name add index index_name(name)
    2. alter table table_name add unique index index_name(name);
"""
import time

for i in range(1):
    start_time = time.time()
    response = llm.run(prompt, generation_kwargs={"temperature": 0.1})
    #print(response['reply'])
    end_time = time.time()
    execution_time = end_time - start_time
    print("执行时间：", execution_time, "秒")
``` 

执行计划: 

```
SWITCH(MySQL执行SELECT语句时崩溃)
Case{
THINKING(可能是由于内存溢出或者查询导致的资源耗尽)
FIND(查看错误日志)
IF (错误日志中有内存溢出或资源耗尽的提示):
    REASON_CONFIRMED(系统资源不足导致崩溃)
    FIND(max_heap_table_size和tmp_table_size参数)
    IF (这两个参数值过小):
        THINKING(增加临时表内存大小可能解决问题)
        RETURN(增大max_heap_table_size和tmp_table_size参数设置)
    ELSE IF (服务器整体内存不足):
        THINKING(需要增加服务器内存或优化内存使用)
        RETURN(增加服务器内存或优化SQL查询)
}
Case{
THINKING(可能是由于查询语句本身的问题)
FIND(获取崩溃时的SELECT语句)
IF (语句过于复杂或者有死循环):
    REASON_CONFIRMED(SELECT语句设计不当)
    THINKING(需要优化查询语句)
    RETURN(优化SELECT语句)
}
Case{
THINKING(可能是由于并发问题)
FIND(查看当前连接数和最大连接数)
IF (当前连接数接近或超过最大连接数):
    REASON_CONFIRMED(并发连接过多)
    THINKING(需要增加最大连接数或优化并发处理)
    RETURN(增大max_connections参数设置或优化并发查询)
}

函数实现:
FIND(查看错误日志)的方法:
    通过`SHOW ENGINE INNODB STATUS`查看InnoDB引擎的状态信息
    或者在服务器日志文件中查找MySQL错误信息

FIND(max_heap_table_size和tmp_table_size参数)的方法:
    使用`SHOW VARIABLES LIKE 'max_heap_table_size'`和`SHOW VARIABLES LIKE 'tmp_table_size'`

FIND(获取崩溃时的SELECT语句)的方法:
    查看错误日志中记录的最后执行的SQL语句

RETURN(增大max_heap_table_size和tmp_table_size参数设置)的方法:
    1. 查看当前配置： `SHOW VARIABLES LIKE 'max_heap_table_size'` 和 `SHOW VARIABLES LIKE 'tmp_table_size'`
    2. 修改当前配置（动态生效）： `SET GLOBAL max_heap_table_size=[new_value];` 和 `SET GLOBAL tmp_table_size=[new_value];`
    3. 如需永久更改，修改my.cnf配置文件并重启MySQL服务

RETURN(优化SELECT语句)的方法:
    1. 分析查询语句，减少JOIN、子查询或使用更有效的索引
    2. 考虑使用LIMIT分页或分解大查询为多个小查询
    3. 根据查询需求优化数据模型或存储过程

RETURN(增大max_connections参数设置)的方法:
    1. 查看当前配置： `SHOW VARIABLES LIKE 'max_connections'`
    2. 修改my.cnf配置文件，增加`max_connections`值
    3. 重启MySQL服务以应用更改
``` 

对涉及optimizer_switch的文档进行评估: 

```
你是一个DBA专家AI, 负责针对人类提出的MySQL数据库问题进行排查计划的生成.
   
人类的最新输入, 是以下 <人类输入> 中的内容:
<人类输入>
MySQL在执行select语句的时候crash，应该如何排查？
</人类输入>
 
AI已经生成了一份排查计划:
<排查计划>
SWITCH(MySQL执行SELECT语句时崩溃)
Case{
THINKING(可能是由于内存溢出或者查询导致的资源耗尽)
FIND(查看错误日志)
IF (错误日志中有内存溢出或资源耗尽的提示):
    REASON_CONFIRMED(系统资源不足导致崩溃)
    FIND(max_heap_table_size和tmp_table_size参数)
    IF (这两个参数值过小):
        THINKING(增加临时表内存大小可能解决问题)
        RETURN(增大max_heap_table_size和tmp_table_size参数设置)
    ELSE IF (服务器整体内存不足):
        THINKING(需要增加服务器内存或优化内存使用)
        RETURN(增加服务器内存或优化SQL查询)
}
Case{
THINKING(可能是由于查询语句本身的问题)
FIND(获取崩溃时的SELECT语句)
IF (语句过于复杂或者有死循环):
    REASON_CONFIRMED(SELECT语句设计不当)
    THINKING(需要优化查询语句)
    RETURN(优化SELECT语句)
}
Case{
THINKING(可能是由于并发问题)
FIND(查看当前连接数和最大连接数)
IF (当前连接数接近或超过最大连接数):
    REASON_CONFIRMED(并发连接过多)
    THINKING(需要增加最大连接数或优化并发处理)
    RETURN(增大max_connections参数设置或优化并发查询)
}

函数实现:
FIND(查看错误日志)的方法:
    通过`SHOW ENGINE INNODB STATUS`查看InnoDB引擎的状态信息
    或者在服务器日志文件中查找MySQL错误信息

FIND(max_heap_table_size和tmp_table_size参数)的方法:
    使用`SHOW VARIABLES LIKE 'max_heap_table_size'`和`SHOW VARIABLES LIKE 'tmp_table_size'`

FIND(获取崩溃时的SELECT语句)的方法:
    查看错误日志中记录的最后执行的SQL语句

RETURN(增大max_heap_table_size和tmp_table_size参数设置)的方法:
    1. 查看当前配置： `SHOW VARIABLES LIKE 'max_heap_table_size'` 和 `SHOW VARIABLES LIKE 'tmp_table_size'`
    2. 修改当前配置（动态生效）： `SET GLOBAL max_heap_table_size=[new_value];` 和 `SET GLOBAL tmp_table_size=[new_value];`
    3. 如需永久更改，修改my.cnf配置文件并重启MySQL服务

RETURN(优化SELECT语句)的方法:
    1. 分析查询语句，减少JOIN、子查询或使用更有效的索引
    2. 考虑使用LIMIT分页或分解大查询为多个小查询
    3. 根据查询需求优化数据模型或存储过程

RETURN(增大max_connections参数设置)的方法:
    1. 查看当前配置： `SHOW VARIABLES LIKE 'max_connections'`
    2. 修改my.cnf配置文件，增加`max_connections`值
    3. 重启MySQL服务以应用更改
</排查计划>
   
   
AI需要学习以下 <数据库服务工单> 中的材料:
<数据库服务工单>

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
</数据库服务工单>
 
你需要判断<数据库服务工单>中的内容, 是否可用于改进现在的<排查计划>, 告诉我你的分析
``` 

结果: 

```
根据<数据库服务工单>中的内容，我们可以对现有的<排查计划>进行一些改进：

1. **添加新的排查步骤**：
   - 在"可能是由于查询语句本身的问题"这个Case中，添加检查是否存在特定类型的查询，如带有EXISTS子查询的语句，特别是那些可能被转换为semi-join操作的查询。因为根据工单，这个问题可能与DuplicateWeedout优化策略有关。
   - 检查MySQL版本，如果低于8.0.24，将其作为可能的原因之一，并建议升级到8.0.24或更高版本。

2. **更新解决方法**：
   - 在"REASON_CONFIRMED(SELECT语句设计不当)"之后，添加关闭DuplicateWeedout优化策略的建议：`SET [GLOBAL|SESSION] optimizer_switch='duplicateweedout=off'`。
   - 强调设置业务表统一的字符集和排序规则，以避免使用DuplicateWeedout策略。
   - 如果无法立即升级MySQL版本，可以提供开启coredump以帮助进一步分析问题的建议。

3. **修改现有排查步骤**：
   - 在"FIND(获取崩溃时的SELECT语句)"之后，添加检查是否存在与工单中提到的类似查询结构的语句。
   - 在"FIND(max_heap_table_size和tmp_table_size参数)"之前，先检查是否与DuplicateWeedout优化策略相关，因为这可能是直接原因，而不是内存资源不足。

结合这些信息，更新后的排查计划应该更加具体地针对工单中描述的问题。
``` 

这个结果从逻辑上来说是没有问题的, 但比起"通用的排查方法", DuplicateWeedout是一种特例, 应当给用户"正道"和"捷径"

# TODO

在make plan的时候, 将"通用方法"和"特例", 或者将 推导过程的必然性/可能性 要求写在Thought中
