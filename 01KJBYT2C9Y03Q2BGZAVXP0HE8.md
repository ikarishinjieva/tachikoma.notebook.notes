---
title: 20221115 - OMA参数列表整理
confluence_page_id: 2130139
created_at: 2022-11-15T10:40:43+00:00
updated_at: 2022-11-17T02:59:52+00:00
---

# 

# 参数解释

### 模式

  - ANALYZE: 兼容性分析: <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000486591>
  - ANALYZE_TOTAL: 整库评估: <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000486605>
  - EVALUATE: 无意义, 会报错
  - REPLAY: 性能评估/回放: <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000486598>  

    - WORKER: 用于分布式回放: <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000486601>
  - DUMP: 导出 OceanBase 数据库的 SQL 语句: <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000486594>
  - PORTRAIT: 无意义, 会报错
  - CONSOLE: 交互式命令行  

    - ![image2022-11-11 0:9:35.png](/assets/01KJBYT2C9Y03Q2BGZAVXP0HE8/image2022-11-11%200%3A9%3A35.png)

### ANALYZE/ANALYZE_TOTAL模式

对ANALYZE工作流的分析

  - ScheduleTask.doRun 或 ScheduleAllModeTask.doRun, 对应mode=ANALYZE 或 ANALYZE_TOTAL
    - configureSourceScanner
      - CollectTask.initTask
        - ScannerFactory.buildScanService
          - ScannerFactory.buildScanService 中, 根据from-type构建了 scan 服务
    - configureConvertTask

      - ConvertTask 进行简单的SQL转换, 例如 MySqlConverter.removeTableComment, coverSqlSecurityDefiner 等
    - configureEvaluateTask

      - 代码位: EvaluateSetServiceImpl.evaluate
      - evaluate-mode=ONLY_TARGET: 
        - onlyTargetEvaluate, 按照目标版本 (OBMYSQL 3.2.3), 解析SQL
      - evaluate-mode=SOURCE_TARGET
        - normalEvaluate, 综合 ONLY_SOURCE 和 ONLY_TARGET
      - evaluate-mode=ONLY_SOURCE
        - onlySourceEvaluate, 按照源数据库版本 (MySQL 5.7), 检查规则(CheckRule)
      - evaluate-mode=ONLY_INSTANCE
        - instanceEvaluate, 通过测试环境OB实例, 进行SQL执行
      - evaluate-mode=APPLICATION_CODE
        - 评估 c/java 程序
    - configurePerformanceEvaluateTask

      - 性能相关的SQL规则, 举例: PerformanceSqlChecker
    - configureTuningTask

      - 调优, 举例: FilterJoinRule (像是半成品, 仅有一个Oracle调优规则)
    - configureReportTask

    - 在以上configure*方法中, 定义了源和目标的topic名, 定义了各个Task的输入和输出流

ANALYZE/ANALYZE_TOTAL 分为几个阶段: 

  - collect: 采集
  - convert: 简单的SQL转换
  - evaluate: SQL的普通规则评估
  - performance evaluate: SQL的性能规则评估
  - tuning: SQL调优 (像是半成品, 仅有一个Oracle调优规则)
  - report: 生成评估报告

各阶段参数: 

  - collect阶段的参数: 
    - from-type, 数据来源: [DB, TEXT, MYBATIS, IBATIS, WCR, COLLECT, OMA, DB_REPLAY, SINGLE, CODE, UNKNOWN, GENERAL_LOG, PORTRAIT] ???
      - from-type = DB  

      - from-type = COLLECT, 仅对 ORACLE 和 DB2LUW 有效, 其他数据库会退化成DB
    - from-type = COLLECT  

      - collect-loop: 有loop时, 不停进行采集, 否则只采集一次
      - collect-during-time: 采集时长 (分钟) ("采集五分钟")
      - collect-interval: 采集的间隔 ("采集五分钟, 每5s采集一次")
      - collect-start-time: 采集时, 查询中的时间起点
      - collect-end-time: 采集时, 查询中的时间终点 ("采集五分钟, 每5s采集一次, 采集从xxx到yyy发生的SQL")
      - collect-filter: 采集时, 查询中的其他条件
    - mapper-config: MyBatis的MapperConfig文件位置
    - objects: 评估对象: [PL, SQL, TABLE, INDEX, VIEW, SEQUENCE, SYNONYM, FUNCTION, PROCEDURE, PACKAGE, TRIGGER, PACKAGE BODY, TYPE, TYPE BODY, DB LINK, MATERIALIZED VIEW, EVENT]
    - parse-thread-count: 分析中同时处理的线程数 (仅有解析WCR文件时, 参数生效)
    - scan-sql: 自定义的采集SQL, 用于采集正在运行的SQL. 如不指定, 每种数据库有默认使用的SQL
    - source-db-*
      - source-db-host

      - source-db-name

      - source-db-password

      - source-db-version

      - source-db-user

      - source-db-type

      - source-db-sid

      - source-db-service-name

      - source-db-port

    - source-file: 来源文件
    - sql-audit-interval: sql audit 间隔 (sql audit的作用?)
    - sql-delimiter: SQL的分隔符 (扫描SQL文件时用)
    - task-interval: 没有意义
    - with-db-cat: 通过DBCat (obtools-dbdiff.jar), dump源库的DDL (作用不明?)  

  - convert阶段的参数: 
    - mask: 脱敏, 没有意义
  - evaluate阶段的参数: 
    - evaluate-mode: 对SQL规则的评估模式  

      - evaluate-mode=ONLY_TARGET: 
        - onlyTargetEvaluate, 按照目标版本 (OBMYSQL 3.2.3), 解析SQL
      - evaluate-mode=SOURCE_TARGET
        - normalEvaluate, 综合 ONLY_SOURCE 和 ONLY_TARGET
      - evaluate-mode=ONLY_SOURCE
        - onlySourceEvaluate, 按照源数据库版本 (MySQL 5.7), 检查规则(CheckRule)
      - evaluate-mode=ONLY_INSTANCE
        - instanceEvaluate, 通过测试环境OB实例, 进行SQL执行
      - evaluate-mode=APPLICATION_CODE
        - 评估 c/java 程序
  - report阶段的参数: 
    - hide-detail: 报告中是否包含语句详情
    - report-root-path
    - store-in-db: 没有意义
  - PerformanceEvaluate 阶段的参数: 
    - performance-mode: 是否对SQL 检查 性能相关的规则  

  - tuning阶段的参数: (属于replay模式?)
    - metadata-from: 数据库元数据的来源 (用作调优依据) [file/jdbc]
    - metadata-file: 如果metadata-from=file, 指定文件路径
    - tuning-mode 是否开启tuning阶段

  - 分析参数
    - analyze-types: [OBJECT, SQL, PORTRAIT, RECOMMENDATION], 仅在ANALYZE_TOTAL 模式下有意义
      - RECOMMENDATION 无意义
      - PORTRAIT: 分析 数据库 快照信息 (DbPortraitAnalyze, 举例: MysqlSpecialTableInfoService) 
      - OBJECT: 分析 数据库 对象
      - SQL: 分析 数据库 SQL
    - extendConfigure: 仅在 Replay Analyze 模式下有意义, 仅有ignore_sqls参数有意义
    - process-thread-count: 分析工作流的并发数 (多少工作流可以同时工作)  

    - schemas: 要分析的schema列表

  

### replay模式

  - 流程 (代码位: ReplayServiceImpl.analyzeSource)
    - COLLECT
      - read service
        - source-type = MySQL: 读取general log
        - source-type = RDS: 读取RDS产生的csv/json
        - 否则, 
          - 如果指定了source-file, 可以读取WCR文件
          - 如果没有指定source-file, 采集OB的SQL?
        - (代码位: CaptureReaderFactory)
      - tuning service
        - 仅对Oracle 11g有效
      - store service
        - 将读取的数据, 附加一些信息, 以json形式写入文件
    - PROCESS: 意义未知  
  

    - SEND
      - 回放到OB
  - replay (单机回放) /ignite (分布式回放) 模式的参数  

    - dump-root-path
    - with-replay-transaction: 回访时支持事务 (OceanBaseTransactionSender)
    - delay-start-time
    - distributed: 分布式 (ignite)
    - filter-exception-sql: 过滤异常SQL (目前仅过滤绑定变量, bind variable)
    - max-parallel: 最大回放线程数 (SEND阶段)
    - parallel-count: 并行数 (STORE阶段)
    - mock: 进行空回放 (SEND阶段)
    - monitor-processor-time: 监控回放进度, 如果超过阈值时间 回放没有进度, 则报错退出
    - gap: 网络延迟. 回访时, 计算时间成本时, 会减去该网络延迟 (作弊用...)
    - nls-format: 设置Oracle NLS format (<https://docs.oracle.com/cd/B19306_01/server.102/b14237/initparams122.htm#REFRN10119>), 没有实际作用
    - ignite相关
      - ignite-replay-table: ignite中保存的WCR表名称 (目前无用途, 表名设置为replay任务名)
      - local-host-address: ignite RemoteAnalyzeTask, 需要使用的本地host?
      - local-http-port: ignite RemoteAnalyzeTask, 需要使用的本地port?
      - node-http-port
      - node-ip-addresses

      - node-multicast-address

      - node-store-path

    - ob-mode: 回放时, OB的模式 Oracle/MySQL
    - query-timeout-sec: 回放时请求超时时间 (SEND阶段)
    - replay-mode: 回放的SQL类型 (SEND阶段): [READ,WRITE,READ_WRITE,PL,ALL]
    - replay-phase: 回放阶段: [COLLECT, PROCESS, SEND]
    - replay-process-name
    - analyse-sample: 采样比例 (COLLECT阶段)
    - replay-sample: 回放比例 (SEND阶段)
    - replay-scale: 回放倍数? (仅影响统计值 SendUtils.calcSendTime)
    - source-file: COLLECT阶段 使用的来源文件
    - source-tenant: COLLECT阶段 使用的租户
    - source-type: COLLECT阶段 使用的OB类型
    - source-version: COLLECT阶段 使用的OB数据库版本, 作用不明
    - split-count: (PROCESS阶段) 分割??
    - 回放目标库
      - target-db-host

      - target-db-password

      - target-db-port

      - target-db-schemas

      - target-db-tenant-cluster

      - target-db-type

      - target-db-user

      - target-db-version

    - warm-up: 回放阶段的预热时间, 只影响统计值计算
    - wcr-version: 读取WCR文件的版本
    - with-reduce: 将采集结果进行聚合 (sort/reduce_summary.json)
    - with-replay-transaction: 回访时使用事务 (OceanBaseTransactionSender)
    - with-display: WCR文件输出到文件? 

  

### DUMP模式

  - oceanbase dump模式的参数
    - dump-root-path
    - dump-type
    - ob-mode: dump时, OB的模式 Oracle/MySQL

### 通用参数

  - 通用参数
    - max-get-connection-time: 连接池的max-wait, 需要连接数据库时使用
    - source-plsql: 没有作用

  

# help信息留档

```
root@ubuntu:/opt/oma-3.3.2# ./bin/start.sh --help
Usage: <main class> [options]
  Options:
    --analyse-sample
      分析抽样，会至少保证每个sqlid有一次采集
      Default: 1.0
    --analyze-table
      ignite中保存的WCR表名称
      Default: <empty string>
    --analyze-types
      需要进行评估的模式，用,分割
      Default: [OBJECT, SQL, PORTRAIT, RECOMMENDATION]
    --cluster-jdbc
      ignite的JDBC接入地址
      Default: 127.0.0.1
    --collect-during-time
      SQL采集运行时间
      Default: 5
    --collect-end-time
      SQL采集结束时间
      Default: 2022-11-13 11:24:10
    --collect-filter
      SQL采集任务过滤条件
      Default: COMMAND_TYPE=3 AND SERVICE != 'SYS$BACKGROUND'
    --collect-interval
      SQL采集时间间隔(秒)
      Default: 30
    --collect-loop

    --collect-start-time
      SQL采集起始时间
      Default: 2022-11-13 10:24:10
    --delay-start-time
      replay启动延迟时间，默认5秒以后启动，单位(秒)
      Default: 5
    --distributed
      是否进行分布式回放
      Default: false
    --dump-root-path
      dump根目录
      Default: /opt/oma-3.3.2/dump/
    --dump-type
      dump类型:[DDL , SQL , ALL]
      Default: ALL
      Possible Values: [DDL, SQL, ALL]
    --evaluate-mode
      评估模式:[NOOP, ONLY_SOURCE, ONLY_TARGET, ONLY_INSTANCE, SOURCE_TARGET,
      SOURCE_INSTANCE, APPLICATION_CODE]
      Possible Values: [NOOP, ONLY_SOURCE, ONLY_TARGET, ONLY_INSTANCE, SOURCE_TARGET, SOURCE_INSTANCE, APPLICATION_CODE]
    --extend-configure
      扩展配置文件，使用java标准配置文件方式解析
      Default: <empty string>
    --filter-exception-sql
      是否过滤异常SQL
      Default: false
    --from-type
      来源类型:[DB , FILE , MYBATIS]
      Possible Values: [DB, TEXT, MYBATIS, IBATIS, WCR, COLLECT, OMA, DB_REPLAY, SINGLE, CODE, UNKNOWN, GENERAL_LOG, PORTRAIT]
    --help

    --hide-detail
      报告中是否包含语句详情
      Default: false
    --ignite-replay-table
      ignite中保存的WCR表名称
      Default: <empty string>
    --local-host-address
      local的地址
      Default: 127.0.0.1
    --local-http-port
      local的http端口
      Default: 40239
    --mapper-config
      MyBatis的MapperConfig文件位置
    --mask
      是否脱敏
      Default: true
    --max-get-connection-time
      获取数据库连接时最大等待时间，单位(毫秒)
      Default: 5000000
    --max-parallel
      最大回放线程数
      Default: 500
    --metadata-file
      元数据信息采集配置文件路径
    --metadata-from
      源数据信息来源
    --mock
      是否进行模拟回放
      Default: false
    --mode
      任务分类,分为dump和analyze,evaluate三种
      Default: ANALYZE
      Possible Values: [DUMP, ANALYZE, EVALUATE, CONSOLE, REPLAY, ANALYZE_TOTAL, WORKER, PORTRAIT]
    --monitor-processor-time
      进程监控时间，默认10分钟
      Default: 10
    --name
      任务名称
    --network-delay
      网络延迟
      Default: 10
    --nls-format
      NLS格式,默认DD-MON-RR
      Default: DD-MON-RR
    --node-http-port
      ignite的http端口
      Default: 26719
    --node-ip-addresses
      ignite接入地址，用,分割
      Default: []
    --node-multicast-address
      ignite的组播地址
      Default: 228.2.6.7
    --node-store-path
      ignite中保存数据的位置
      Default: /opt/oma-3.3.2/work/db
    --ob-mode
      ob模式：ORACLE或者MYSQL
      Default: ORACLE
    --objects
      评估对象，用,分割
      Default: [PL, SQL, TABLE, INDEX, VIEW, SEQUENCE, SYNONYM, FUNCTION, PROCEDURE, PACKAGE, TRIGGER, PACKAGE BODY, TYPE, TYPE BODY, DB LINK, MATERIALIZED VIEW, EVENT]
    --parallel-count
      replay中并行处理数量
      Default: 5
    --parse-thread-count
      分析中同时处理的线程数，默认10
      Default: 10
    --performance-mode
      是否开启静态性能评估
      Default: false
    --process-thread-count
      处理线程数量
      Default: 1
    --query-timeout-sec
      请求的超时时间，秒
      Default: 30
    --replay-mode
      回放模式:[READ,WRITE,READ_WRITE,PL,ALL]
      Default: READ
      Possible Values: [READ, WRITE, READ_WRITE, PL, ALL]
    --replay-phase
      回放阶段:[COLLECT , PROCESS , SEND]
      Possible Values: [COLLECT, PROCESS, SEND]
    --replay-process-name
      replay处理任务的名称
      Default: sort
    --replay-sample
      回放抽样
      Default: 1.0
    --replay-scale
      回放倍数
      Default: 1.0
    --report-root-path
      report根目录
      Default: /opt/oma-3.3.2/report/
    --scan-sql
      采集待评估语句执行的SQL
    --schemas
      schema，用,分割
      Default: [DEFAULT]
    --source-db-host
      来源数据库地址
    --source-db-name
      dbname
    --source-db-password
      来源数据库密码
    --source-db-port
      来源数据库端口
    --source-db-service-name
      来源数据库service-name
    --source-db-sid
      来源数据库sid
    --source-db-type
      来源数据库类型
      Default: ORACLE
    --source-db-user
      来源数据库用户名
    --source-db-version
      来源数据库版本
      Default: 11g
    --source-file
      来源文件路径
    --source-plsql
      待评估的plsql语句
      Default: <empty string>
    --source-tenant
      采集的租户
      Default: <empty string>
    --source-type
      来源文件类型
      Default: ORACLE
    --source-version
      来源数据库版本
    --split-count
      replay中切割的数量
      Default: 0
    --sql-audit-interval
      SQL AUDIT的扫描间隔
      Default: 60
    --sql-delimiter
      sql语句分隔符
    --store-in-db
      是否输出详细报告
      Default: false
    --target-db-host
      目标数据库地址
    --target-db-password
      来源数据库密码
    --target-db-port
      目标数据库端口
    --target-db-schemas
      schema，用,分割
      Default: [DEFAULT]
    --target-db-tenant-cluster
      目标数据库租户和集群
      Default: <empty string>
    --target-db-type
      目标数据库类型
      Default: OBORACLE
    --target-db-user
      来源数据库用户名
    --target-db-version
      目标数据库版本
      Default: 3.2.3.0
    --task-interval
      SQL采集任务运行间隔(秒)
      Default: -1
    --tuning-mode
      是否开启sql调优
      Default: false
    --warm-up
      预热时间（秒）
      Default: 0
    --wcr-version
      wcr文件的版本
      Default: <empty string>
    --with-db-cat
      是否采用dbcat
      Default: false
    --with-display
      输出详情到文件
      Default: false
    --with-reduce
      收集完以后是否合并
      Default: false
    --with-replay-transaction
      是否进行事务回放，默认不进行
      Default: false
    -D
      其它辅助参数,使用,分割参数，使用:分割key和value,特殊字符需要转义
      Syntax: -Dkey=value
      Default: {}

root@ubuntu:/opt/oma-3.3.2#
```
