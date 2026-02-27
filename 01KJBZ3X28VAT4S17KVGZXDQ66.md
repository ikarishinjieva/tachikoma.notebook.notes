---
title: 20231115 - Oracle logminer学习
confluence_page_id: 2589475
created_at: 2023-11-15T05:36:18+00:00
updated_at: 2023-11-15T06:24:02+00:00
---

# 官方文档

  - <https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/DBMS_LOGMNR.html#GUID-41730EFC-C6CA-423E-834B-3E0E643346C3>
    - 为什么OMS没使用 COMMITTED_DATA_ONLY参数
      - COMMITTED_DATA_ONLY的作用: logminer机制 仅返回已提交事务, 并按照事务提交顺序进行  

    - DDL_DICT_TRACKING: 作用未知  

  

# 分析OMS使用Logminer的行为:

```
[arthas@14680]$ watch oracle.jdbc.driver.T4CConnection setExecutingRPCSQL '{@java.lang.System@currentTimeMillis(), "(Oracle SQL)", @java.lang.Thread@currentThread().name, params[0]}' '(params[0] != null) && (@java.lang.Thread@currenead().name != "sysdate-synchronizer") && (@java.lang.Thread@currentThread().name != "scn-synchronizer")'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 1339 ms, listenerId: 25
method=oracle.jdbc.driver.T4CConnection.setExecutingRPCSQL location=AtExit
ts=2023-11-15 14:07:04; [cost=0.067067ms] result=@ArrayList[
    @Long[1700028424035],
    @String[(Oracle SQL)],
    @String[LogEntryPrimaryFetcher#1],
    @String[SELECT V$LOGFILE.MEMBER NAME, V$LOG.THREAD# THREAD_NUMBER, V$LOG.SEQUENCE# SEQUENCE_NUMBER, V$LOG.FIRST_CHANGE# FIRST_CHANGE_NUMBER, LEAD(V$LOG.FIRST_CHANGE#, 1, 281474976710655) OVER (ORDER BY V$LOG.SEQUENCE#) NEXT_CHANGE_NUMBER, TO_CHAR(V$LOG.FIRST_TIME, 'YYYY-MM-DD HH24:MI:SS') FIRST_TIME, TO_CHAR(LEAD(V$LOG.FIRST_TIME, 1, NULL) OVER (ORDER BY V$LOG.SEQUENCE#), 'YYYY-MM-DD HH24:MI:SS') NEXT_TIME, 0 BLOCK_SIZE, V$LOG.BYTES BYTES, V$LOG.GROUP# GROUP_NUMBER, V$LOG.MEMBERS MEMBERS, V$LOG.ARCHIVED ARCHIVED, V$LOG.STATUS STATUS FROM V$LOG, V$LOGFILE WHERE (V$LOG.STATUS = 'CURRENT' OR V$LOG.STATUS = 'ACTIVE' OR V$LOG.STATUS = 'INACTIVE') AND V$LOG.GROUP# = V$LOGFILE.GROUP# AND V$LOG.THREAD# = :1  ORDER BY V$LOG.SEQUENCE#],
]
method=oracle.jdbc.driver.T4CConnection.setExecutingRPCSQL location=AtExit
ts=2023-11-15 14:07:04; [cost=0.016619ms] result=@ArrayList[
    @Long[1700028424041],
    @String[(Oracle SQL)],
    @String[LogEntryPrimaryFetcher#1],
    @String[SELECT * FROM (SELECT ROWNUM RN, SCN, OPERATION_CODE, SEG_TYPE, TO_CHAR(TIMESTAMP, 'YYYY-MM-DD HH24:MI:SS') TIMESTAMP, XID, ROW_ID, SQL_REDO, CSF, RBASQN, RBABLK, RBABYTE, TABLE_NAME, SEG_OWNER, THREAD#, ROLLBACK, SEG_NAME, INFO, DATA_OBJ#, RS_ID FROM V$LOGMNR_CONTENTS WHERE SCN >= 1570628) WHERE RN >= :1 ],
]
method=oracle.jdbc.driver.T4CConnection.setExecutingRPCSQL location=AtExit
ts=2023-11-15 14:07:24; [cost=0.031062ms] result=@ArrayList[
    @Long[1700028444767],
    @String[(Oracle SQL)],
    @String[LogEntryPrimaryFetcher#1],
    @String[SELECT V$LOGFILE.MEMBER NAME, V$LOG.THREAD# THREAD_NUMBER, V$LOG.SEQUENCE# SEQUENCE_NUMBER, V$LOG.FIRST_CHANGE# FIRST_CHANGE_NUMBER, LEAD(V$LOG.FIRST_CHANGE#, 1, 281474976710655) OVER (ORDER BY V$LOG.SEQUENCE#) NEXT_CHANGE_NUMBER, TO_CHAR(V$LOG.FIRST_TIME, 'YYYY-MM-DD HH24:MI:SS') FIRST_TIME, TO_CHAR(LEAD(V$LOG.FIRST_TIME, 1, NULL) OVER (ORDER BY V$LOG.SEQUENCE#), 'YYYY-MM-DD HH24:MI:SS') NEXT_TIME, 0 BLOCK_SIZE, V$LOG.BYTES BYTES, V$LOG.GROUP# GROUP_NUMBER, V$LOG.MEMBERS MEMBERS, V$LOG.ARCHIVED ARCHIVED, V$LOG.STATUS STATUS FROM V$LOG, V$LOGFILE WHERE (V$LOG.STATUS = 'CURRENT' OR V$LOG.STATUS = 'ACTIVE' OR V$LOG.STATUS = 'INACTIVE') AND V$LOG.GROUP# = V$LOGFILE.GROUP# AND V$LOG.THREAD# = :1  ORDER BY V$LOG.SEQUENCE#],
]
method=oracle.jdbc.driver.T4CConnection.setExecutingRPCSQL location=AtExit
ts=2023-11-15 14:07:24; [cost=0.015757ms] result=@ArrayList[
    @Long[1700028444787],
    @String[(Oracle SQL)],
    @String[LogEntryPrimaryFetcher#1],
    @String[SELECT * FROM (SELECT ROWNUM RN, SCN, OPERATION_CODE, SEG_TYPE, TO_CHAR(TIMESTAMP, 'YYYY-MM-DD HH24:MI:SS') TIMESTAMP, XID, ROW_ID, SQL_REDO, CSF, RBASQN, RBABLK, RBABYTE, TABLE_NAME, SEG_OWNER, THREAD#, ROLLBACK, SEG_NAME, INFO, DATA_OBJ#, RS_ID FROM V$LOGMNR_CONTENTS WHERE SCN >= 1570628) WHERE RN >= :1 ],
]
``` 

  

发现: 对于相对空闲的压力, 每20秒, OMS会从Oracle的LOGMNR机制轮询一轮

通过arthas stack命令, 获取调用堆栈, 两个SQL的堆栈分别为: 

```
ts=2023-11-15 14:16:23;thread_name=LogEntryPrimaryFetcher#1;id=41;is_daemon=true;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@764c12b6
    @oracle.jdbc.driver.T4CConnection.setExecutingRPCSQL()
        at oracle.jdbc.driver.T4CTTIfun.receive(T4CTTIfun.java:889)
        at oracle.jdbc.driver.T4CTTIfun.doRPC(T4CTTIfun.java:298)
        at oracle.jdbc.driver.T4C8Oall.doOALL(T4C8Oall.java:497)
        at oracle.jdbc.driver.T4CPreparedStatement.doOall8(T4CPreparedStatement.java:151)
        at oracle.jdbc.driver.T4CPreparedStatement.executeForDescribe(T4CPreparedStatement.java:936)
        at oracle.jdbc.driver.OracleStatement.prepareDefineBufferAndExecute(OracleStatement.java:1171)
        at oracle.jdbc.driver.OracleStatement.executeMaybeDescribe(OracleStatement.java:1100)
        at oracle.jdbc.driver.OracleStatement.executeSQLSelect(OracleStatement.java:1425)
        at oracle.jdbc.driver.OracleStatement.doExecuteWithTimeout(OracleStatement.java:1308)
        at oracle.jdbc.driver.OraclePreparedStatement.executeInternal(OraclePreparedStatement.java:3745)
        at oracle.jdbc.driver.OraclePreparedStatement.executeQuery(OraclePreparedStatement.java:3854)
        at oracle.jdbc.driver.OraclePreparedStatementWrapper.executeQuery(OraclePreparedStatementWrapper.java:1097)
        at com.taobao.drc.logminer.util.JdbcUtil.executeMultiRowsQuery(JdbcUtil.java:91)
        at com.taobao.drc.logminer.util.LogFileUtil.getRedoLogFiles(LogFileUtil.java:204)
        at com.taobao.drc.logminer.util.LogFileUtil.getRedoLogFile(LogFileUtil.java:244)
        at com.taobao.drc.logminer.fetcher.LogMiner.stillCurrentActiveLogFile(LogMiner.java:105)
        at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadNonArchLogEntriesWithValidationExceptionResume(LogMinerUtil.java:245)
        at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadNonArchLogEntriesWithValidationExceptionResume(LogMinerUtil.java:222)
        at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadNonArchLogEntriesWithBreakpointResume(LogMinerUtil.java:207)
        at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadRedoLogEntries(LogMinerUtil.java:148)
        at com.taobao.drc.logminer.fetcher.LogEntryPrimaryFetcher.fetchLogEntriesFromRedoLogFile(LogEntryPrimaryFetcher.java:67)
        at com.taobao.drc.logminer.fetcher.LogEntryPrimaryFetcher.fetchNonArchLogEntriesOneRound(LogEntryPrimaryFetcher.java:59)
        at com.taobao.drc.logminer.fetcher.LogEntryFetcher.notOnlyFetchArchLogEntriesLoop(LogEntryFetcher.java:147)
        at com.taobao.drc.logminer.fetcher.LogEntryFetcher.fetchLogEntriesLoop(LogEntryFetcher.java:116)
        at com.taobao.drc.logminer.fetcher.LogEntryFetcher.run(LogEntryFetcher.java:91)
``` 

```
ts=2023-11-15 14:16:23;thread_name=LogEntryPrimaryFetcher#1;id=41;is_daemon=true;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@764c12b6
    @oracle.jdbc.driver.T4CConnection.setExecutingRPCSQL()
        at oracle.jdbc.driver.T4C8Oall.setExecutingSQLBeforeCall(T4C8Oall.java:683)
        at oracle.jdbc.driver.T4C8Oall.doOALL(T4C8Oall.java:496)
        at oracle.jdbc.driver.T4CPreparedStatement.doOall8(T4CPreparedStatement.java:151)
        at oracle.jdbc.driver.T4CPreparedStatement.executeForDescribe(T4CPreparedStatement.java:936)
        at oracle.jdbc.driver.OracleStatement.prepareDefineBufferAndExecute(OracleStatement.java:1171)
        at oracle.jdbc.driver.OracleStatement.executeMaybeDescribe(OracleStatement.java:1100)
        at oracle.jdbc.driver.OracleStatement.executeSQLSelect(OracleStatement.java:1425)
        at oracle.jdbc.driver.OracleStatement.doExecuteWithTimeout(OracleStatement.java:1308)
        at oracle.jdbc.driver.OraclePreparedStatement.executeInternal(OraclePreparedStatement.java:3745)
        at oracle.jdbc.driver.OraclePreparedStatement.executeQuery(OraclePreparedStatement.java:3854)
        at oracle.jdbc.driver.OraclePreparedStatementWrapper.executeQuery(OraclePreparedStatementWrapper.java:1097)
        at com.taobao.drc.logminer.fetcher.LogMiner.loadLogEntries(LogMiner.java:167)
        at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadNonArchLogEntriesWithValidationExceptionResume(LogMinerUtil.java:254)
        at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadNonArchLogEntriesWithValidationExceptionResume(LogMinerUtil.java:222)
        at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadNonArchLogEntriesWithBreakpointResume(LogMinerUtil.java:207)
        at com.taobao.drc.logminer.fetcher.LogMinerUtil.loadRedoLogEntries(LogMinerUtil.java:148)
        at com.taobao.drc.logminer.fetcher.LogEntryPrimaryFetcher.fetchLogEntriesFromRedoLogFile(LogEntryPrimaryFetcher.java:67)
        at com.taobao.drc.logminer.fetcher.LogEntryPrimaryFetcher.fetchNonArchLogEntriesOneRound(LogEntryPrimaryFetcher.java:59)
        at com.taobao.drc.logminer.fetcher.LogEntryFetcher.notOnlyFetchArchLogEntriesLoop(LogEntryFetcher.java:147)
        at com.taobao.drc.logminer.fetcher.LogEntryFetcher.fetchLogEntriesLoop(LogEntryFetcher.java:116)
        at com.taobao.drc.logminer.fetcher.LogEntryFetcher.run(LogEntryFetcher.java:91)
``` 

时间间隔的代码: 

![image2023-11-15 14:23:55.png](/assets/01KJBZ3X28VAT4S17KVGZXDQ66/image2023-11-15%2014%3A23%3A55.png)

  

# 技巧: arthas获取当前进程中的所有oracle连接正在执行的SQL

```
vmtool --action getInstances --className oracle.jdbc.driver.T4CConnection --express 'instances.{#this.executingRPCSQL}'
``` 

  

示例: 

```
[arthas@14680]$ vmtool --action getInstances --className oracle.jdbc.driver.T4CConnection --express 'instances.{#this.executingRPCSQL}'
@ArrayList[
    null,
    @String[SELECT * FROM (SELECT ROWNUM RN, SCN, OPERATION_CODE, SEG_TYPE, TO_CHAR(TIMESTAMP, 'YYYY-MM-DD HH24:MI:SS') TIMESTAMP, XID, ROW_ID, SQL_REDO, CSF, RBASQN, RBABLK, RBABYTE, TABLE_NAME, SEG_OWNER, THREAD#, ROLLBACK, SEG_NAME, INFO, DATA_OBJ#, RS_ID FROM V$LOGMNR_CONTENTS WHERE SCN >= 1570628) WHERE RN >= :1 ],
    null,
    null,
    null,
    null,
]
``` 

  

# 技巧: arthas获取当前进程中, 执行SQL的动作: 

```
[arthas@14680]$ watch oracle.jdbc.driver.T4CConnection setExecutingRPCSQL '{@java.lang.System@currentTimeMillis(), "(Oracle SQL)", @java.lang.Thread@currentThread().name, params[0]}' '(params[0] != null) && (@java.lang.Thread@currenead().name != "sysdate-synchronizer") && (@java.lang.Thread@currentThread().name != "scn-synchronizer")'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 1339 ms, listenerId: 25
method=oracle.jdbc.driver.T4CConnection.setExecutingRPCSQL location=AtExit
ts=2023-11-15 14:07:04; [cost=0.067067ms] result=@ArrayList[
    @Long[1700028424035],
    @String[(Oracle SQL)],
    @String[LogEntryPrimaryFetcher#1],
    @String[SELECT V$LOGFILE.MEMBER NAME, V$LOG.THREAD# THREAD_NUMBER, V$LOG.SEQUENCE# SEQUENCE_NUMBER, V$LOG.FIRST_CHANGE# FIRST_CHANGE_NUMBER, LEAD(V$LOG.FIRST_CHANGE#, 1, 281474976710655) OVER (ORDER BY V$LOG.SEQUENCE#) NEXT_CHANGE_NUMBER, TO_CHAR(V$LOG.FIRST_TIME, 'YYYY-MM-DD HH24:MI:SS') FIRST_TIME, TO_CHAR(LEAD(V$LOG.FIRST_TIME, 1, NULL) OVER (ORDER BY V$LOG.SEQUENCE#), 'YYYY-MM-DD HH24:MI:SS') NEXT_TIME, 0 BLOCK_SIZE, V$LOG.BYTES BYTES, V$LOG.GROUP# GROUP_NUMBER, V$LOG.MEMBERS MEMBERS, V$LOG.ARCHIVED ARCHIVED, V$LOG.STATUS STATUS FROM V$LOG, V$LOGFILE WHERE (V$LOG.STATUS = 'CURRENT' OR V$LOG.STATUS = 'ACTIVE' OR V$LOG.STATUS = 'INACTIVE') AND V$LOG.GROUP# = V$LOGFILE.GROUP# AND V$LOG.THREAD# = :1  ORDER BY V$LOG.SEQUENCE#],
]
method=oracle.jdbc.driver.T4CConnection.setExecutingRPCSQL location=AtExit
ts=2023-11-15 14:07:04; [cost=0.016619ms] result=@ArrayList[
    @Long[1700028424041],
    @String[(Oracle SQL)],
    @String[LogEntryPrimaryFetcher#1],
    @String[SELECT * FROM (SELECT ROWNUM RN, SCN, OPERATION_CODE, SEG_TYPE, TO_CHAR(TIMESTAMP, 'YYYY-MM-DD HH24:MI:SS') TIMESTAMP, XID, ROW_ID, SQL_REDO, CSF, RBASQN, RBABLK, RBABYTE, TABLE_NAME, SEG_OWNER, THREAD#, ROLLBACK, SEG_NAME, INFO, DATA_OBJ#, RS_ID FROM V$LOGMNR_CONTENTS WHERE SCN >= 1570628) WHERE RN >= :1 ],
]
method=oracle.jdbc.driver.T4CConnection.setExecutingRPCSQL location=AtExit
ts=2023-11-15 14:07:24; [cost=0.031062ms] result=@ArrayList[
    @Long[1700028444767],
    @String[(Oracle SQL)],
    @String[LogEntryPrimaryFetcher#1],
    @String[SELECT V$LOGFILE.MEMBER NAME, V$LOG.THREAD# THREAD_NUMBER, V$LOG.SEQUENCE# SEQUENCE_NUMBER, V$LOG.FIRST_CHANGE# FIRST_CHANGE_NUMBER, LEAD(V$LOG.FIRST_CHANGE#, 1, 281474976710655) OVER (ORDER BY V$LOG.SEQUENCE#) NEXT_CHANGE_NUMBER, TO_CHAR(V$LOG.FIRST_TIME, 'YYYY-MM-DD HH24:MI:SS') FIRST_TIME, TO_CHAR(LEAD(V$LOG.FIRST_TIME, 1, NULL) OVER (ORDER BY V$LOG.SEQUENCE#), 'YYYY-MM-DD HH24:MI:SS') NEXT_TIME, 0 BLOCK_SIZE, V$LOG.BYTES BYTES, V$LOG.GROUP# GROUP_NUMBER, V$LOG.MEMBERS MEMBERS, V$LOG.ARCHIVED ARCHIVED, V$LOG.STATUS STATUS FROM V$LOG, V$LOGFILE WHERE (V$LOG.STATUS = 'CURRENT' OR V$LOG.STATUS = 'ACTIVE' OR V$LOG.STATUS = 'INACTIVE') AND V$LOG.GROUP# = V$LOGFILE.GROUP# AND V$LOG.THREAD# = :1  ORDER BY V$LOG.SEQUENCE#],
]
method=oracle.jdbc.driver.T4CConnection.setExecutingRPCSQL location=AtExit
ts=2023-11-15 14:07:24; [cost=0.015757ms] result=@ArrayList[
    @Long[1700028444787],
    @String[(Oracle SQL)],
    @String[LogEntryPrimaryFetcher#1],
    @String[SELECT * FROM (SELECT ROWNUM RN, SCN, OPERATION_CODE, SEG_TYPE, TO_CHAR(TIMESTAMP, 'YYYY-MM-DD HH24:MI:SS') TIMESTAMP, XID, ROW_ID, SQL_REDO, CSF, RBASQN, RBABLK, RBABYTE, TABLE_NAME, SEG_OWNER, THREAD#, ROLLBACK, SEG_NAME, INFO, DATA_OBJ#, RS_ID FROM V$LOGMNR_CONTENTS WHERE SCN >= 1570628) WHERE RN >= :1 ],
]
```
