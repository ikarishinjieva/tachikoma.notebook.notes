---
title: 20221022 - 旧: Clickhouse+学习笔记
confluence_page_id: 1146957
created_at: 2021-06-08T03:46:18+00:00
updated_at: 2024-04-07T08:26:54+00:00
---

# Clickhouse 学习笔记

# 目录

  - [Clickhouse学习笔记-目录]
  - [Clickhouse学习笔记-业务场景/数据集]
  - [Clickhouse学习笔记-文档]
  - [Clickhouse学习笔记-执行计划explainplan格式]
  - [Clickhouse学习笔记-执行计划explainpipeline格式]
  - [Clickhouse学习笔记-异常现象-1]
  - [Clickhouse学习笔记-知识框架]
  - [Clickhouse学习笔记-MaterializedMySQL]
  - [Clickhouse学习笔记-TODO]

# 业务场景/数据集

  - [{+}](<https://amplab.cs.berkeley.edu/benchmark/>)<https://amplab.cs.berkeley.edu/benchmark/+>
  - [Apache Hadoop Benchmarking](<https://issues.apache.org/jira/browse/MAPREDUCE-3561>) \- micro-benchmarks for testing Hadoop performances.
  - [Berkeley SWIM Benchmark](<https://github.com/SWIMProjectUCB/SWIM/wiki>) \- real-world big data workload benchmark.
  - [Intel HiBench](<https://github.com/intel-hadoop/HiBench>) \- a Hadoop benchmark suite.
  - [PUMA Benchmarking](<https://issues.apache.org/jira/browse/MAPREDUCE-5116>) \- benchmark suite for MapReduce applications.
  - [Yahoo Gridmix3](<http://yahoohadoop.tumblr.com/post/98294079296/gridmix3-emulating-production-workload-for>) \- Hadoop cluster benchmarking from Yahoo engineer team.
  - [Deeplearning4j Benchmarks](<https://github.com/deeplearning4j/dl4j-benchmark>)
  - [{+}](<http://www.olapcouncil.org/research/spec1.htm>)<http://www.olapcouncil.org/research/spec1.htm+>
  - benchmark综述: [{+}](<https://arxiv.org/pdf/1701.08634.pdf>)<https://arxiv.org/pdf/1701.08634.pdf+>
  - [{+}](<https://github.com/Microsoft/sql-server-samples/releases/tag/iot-smart-grid-v1.0>)<https://github.com/Microsoft/sql-server-samples/releases/tag/iot-smart-grid-v1.0+>
  - [{+}](<https://github.com/microsoft/sql-server-samples/tree/master/samples/databases/adventure-works>)<https://github.com/microsoft/sql-server-samples/tree/master/samples/databases/adventure-works+>
  - [{+}](<https://docs.cambridgesemantics.com/anzograph/v2.2/userdoc/tickit.htm#marketing>)<https://docs.cambridgesemantics.com/anzograph/v2.2/userdoc/tickit.htm#marketing+>
  - [{+}](<https://docs.cambridgesemantics.com/anzograph/v2.2/userdoc/ghib.htm>)<https://docs.cambridgesemantics.com/anzograph/v2.2/userdoc/ghib.htm+>
  - [{+}](<https://docs.cambridgesemantics.com/anzograph/v2.2/userdoc/cypher.htm>)<https://docs.cambridgesemantics.com/anzograph/v2.2/userdoc/cypher.htm+>
  - [{+}](<https://clickhouse.tech/docs/en/getting-started/example-datasets/metrica/>)<https://clickhouse.tech/docs/en/getting-started/example-datasets/metrica/+>
  - [{+}](<https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/>)<https://clickhouse.tech/docs/en/getting-started/example-datasets/star-schema/+>
  - [{+}](<https://clickhouse.tech/docs/en/getting-started/example-datasets/brown-benchmark/>)<https://clickhouse.tech/docs/en/getting-started/example-datasets/brown-benchmark/+>
  - [{+}](<https://clickhouse.tech/docs/en/getting-started/example-datasets/nyc-taxi/>)<https://clickhouse.tech/docs/en/getting-started/example-datasets/nyc-taxi/+>
  - [{+}](<https://clickhouse.tech/docs/en/getting-started/example-datasets/ontime/>)<https://clickhouse.tech/docs/en/getting-started/example-datasets/ontime/+>

# 文档

  - 文档: [{+}](<https://www.slideshare.net/Altinity/clickhouse-deep-dive-by-aleksei-milovidov>)<https://www.slideshare.net/Altinity/clickhouse-deep-dive-by-aleksei-milovidov+>
    - 稀疏索引
      - 默认粒度为8192行, 作为一个块, 每次读取一整块
      - 非唯一索引
      - point select 性能差
    - Merge tree 数据更新步骤
      - ![worddavbdcabbd1a394b8c59ccc6064edfd5fd1.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavbdcabbd1a394b8c59ccc6064edfd5fd1.png)
      - ![worddavcfcf9db70b8eebae7fe04870402328f4.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavcfcf9db70b8eebae7fe04870402328f4.png)
      - ![worddav6b651e4c7819e5b6e9fb01cc24691887.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav6b651e4c7819e5b6e9fb01cc24691887.png)
      - ![worddavd2e750488032098c655e66b3ac171542.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavd2e750488032098c655e66b3ac171542.png)
      - Merge 操作在后台进行
    - Distributed table是逻辑view, 实际数据存储在local table
    - ReplicatedMergeTree 可实现异步主主复制

  - 文档: [{+}](<https://www.slideshare.net/Altinity/clickhouse-data-warehouse-101-the-first-billion-rows-by-alexander-zaitsev-and-robert-hodges-altinity>)<https://www.slideshare.net/Altinity/clickhouse-data-warehouse-101-the-first-billion-rows-by-alexander-zaitsev-and-robert-hodges-altinity+>
    - 性能优化手段: 
      - SETTING max_threads = 8
      - 使用合适的数据类型
      - MaterializedView 与 SummingMergeTree 联合
      - LowCardinality 数据类型修饰, 使用字典
      - Array 用于1-N关系
      - 优化JOIN操作, 先GROUP BY, 后JOIN
        - ![worddavcf769bf104c3b7d853c82846254fb696.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavcf769bf104c3b7d853c82846254fb696.png)
        - ![worddav0b31cda76709106873717eda71a7442a.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav0b31cda76709106873717eda71a7442a.png)

  - 文档: [{+}](<https://www.slideshare.net/Altinity/high-performance-high-reliability-data-loading-on-clickhouse>)<https://www.slideshare.net/Altinity/high-performance-high-reliability-data-loading-on-clickhouse+>, 加速INSERT的方法
    - buffer tables: 
      - 使用buffer engine, 缓存小量INSERT
      - 缺点: 经不起重启
        - 通过in_memory_parts_enable_wal解决
      - ![worddav33a977ce5827004057855d5fd1f98bfd.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav33a977ce5827004057855d5fd1f98bfd.png)
    - MergeTree compact格式
      - wide格式, 每列一个文件
      - compact格式, 一个文件多列, 相关参数: 
        - min_bytes_for_wide_part
        - min_rows_for_wide_part
        - 减少文件系统消耗, 适合小量高频INSERT
    - clickhouse没有事务性, 大量INSERT无法保持原子性
      - 通过Temp中转
        - ![worddav5efeacdf5b2d2a784d15fcc1c7ec3ca3.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav5efeacdf5b2d2a784d15fcc1c7ec3ca3.png)
      - 持久化相关参数
        - ![worddaveeb3b78aed37b581c2d2abbc56d0d86b.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddaveeb3b78aed37b581c2d2abbc56d0d86b.png)
    - 最佳实践
      - ![worddavc779d592f3223c222c091947f3ddcdd7.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavc779d592f3223c222c091947f3ddcdd7.png)
    - 去重
      - Clickhouse没有唯一键约束
      - 块级别去重: Replicated表 通过zk同步复制元数据, 其中包括block的hash, 可用于去重. 相关参数: 
        - replicated_deduplication_window
        - replicated_deduplication_window_seconds
        - deduplicate_blocks_in_dependent_materialized_view
      - ReplacingMergeTree 可达成最终去重 (在OPTIMIZE FINAL或其他机制进行去重)
      - 逻辑去重方式: 
        - ![worddav69cc1a4a0837d82161c47c6ffce12df1.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav69cc1a4a0837d82161c47c6ffce12df1.png)
      - OPTIMIZE DEDUPLICATE 命令
  - 文档: [{+}](<https://www.slideshare.net/Altinity/clickhouse-query-performance-tips-and-tricks-by-robert-hodges-altinity-ceo>)<https://www.slideshare.net/Altinity/clickhouse-query-performance-tips-and-tricks-by-robert-hodges-altinity-ceo+>
    - partition by与order by的区别: 
      - ![worddavc05c096da7f3253d80598997ed063b3c.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavc05c096da7f3253d80598997ed063b3c.png)
    - 如何使用trace log: 
      - clickhouse-client --send_logs_level=trace
    - PREWHERE与WHERE的区别
      - PREWHERE先读出where相关的列, 找到匹配后再回表查询, 这样读出的数据量小
      - WHERE将where相关的列和select相关的列一起读出, 这样对于where不命中的行, select相关的部分的读取就浪费了
      - PREWHERE可以自动触发, 在trace log中有相关记录
    - clickhouse会遵循SQL定义的顺序执行, 需要调整SQL使得其扫描数据最少
      - 举例: 原始SQL先Join后Group by, 修改为先Group by后left join
      - ![worddava6dcd41c2c4d1ffe9232b23a655a894f.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddava6dcd41c2c4d1ffe9232b23a655a894f.png)
    - general log: 
      - set log_queries=1
      - select * from system.query_log
    - 调整数据分布结构: 
      - 确认分区的数量是否合适
        - 分区数量维持在几百个即可 (partition by可以使用函数)
      - 优化主键索引和排序
        - ![worddava50522c0d1fbee40cc9d9189a338c787.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddava50522c0d1fbee40cc9d9189a338c787.png)
        - ORDER BY定义了PARTITION内部的排序, 在ORDER BY中加上PARTITION的key, 使得索引hash计算的数据更大, 离散度更好 ??
        - 粒度的选择: 
          - 大粒度使得索引更小
          - 小粒度使得索引选择度更好 (可跳过更多地不必要的索引页)
      - 添加合适的索引, 使得查询条件可以更多地跳过不必要的索引页
        - ![worddav7d2e58189345c82ca586a3e1d415e598.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav7d2e58189345c82ca586a3e1d415e598.png)
      - 调整encoding, 以改变数据量的大小, 从而影响性能
        - ![worddav7ed7bc725ca24d3984500b1d52d4cb76.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav7ed7bc725ca24d3984500b1d52d4cb76.png)
      - 使用物化视图
        - ![worddav7f84d41f9cb66e7f0150f9a48999f1c4.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav7f84d41f9cb66e7f0150f9a48999f1c4.png)
      - 其他技巧
        - 用 external directionary 代替 JOIN ???
        - 如果是统计结果, 使用 SAMPLE 子句
    - 监控诊断
      - 查看分区信息
        - ![worddavc22481b318cd73415e2cfd47cf841813.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavc22481b318cd73415e2cfd47cf841813.png)
      - 查看数据大小
        - ![worddava5e1ecc2aa8cdc21baf1d027fdcfa294.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddava5e1ecc2aa8cdc21baf1d027fdcfa294.png)
  - [{+}](<https://presentations.clickhouse.tech/bdtc_2019/#9>)<https://presentations.clickhouse.tech/bdtc_2019+>: The Secrets of ClickHouse Performance Optimizations
    - 文档强调了Clickhouse的性能来源: 根据业务类型决定细节技术, 包括 substring match/hash table等
    - ![worddavd2c404fe16bdb300a8818b3cefcb75ad.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavd2c404fe16bdb300a8818b3cefcb75ad.png)![worddav59a8fe57a02c71880099e46f821131a7.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav59a8fe57a02c71880099e46f821131a7.png)
      - 根据底层能力, 计算最大能力
  - [{+}](<https://presentations.clickhouse.tech/database_saturday_2020/#26>)<https://presentations.clickhouse.tech/database_saturday_2020/#26+>
    - 提到了一种优化技术 (生产中未使用)
      - 当iTLB (页表) 命中率降低时, 诊断其由于代码段增大引起, 需要将代码放到 大页 中, 但透明大页机制仅限于匿名内存, 不包括代码段
  - [{+}](<https://presentations.clickhouse.tech/dataops_2019/#24>)<https://presentations.clickhouse.tech/dataops_2019/#24+>
    - 对clickhouse诊断性能的工具: 
      - select elapsed, formatReadableSize(memory_usage) AS mem, initial_user, formatReadableSize(read_bytes) AS bytes, substring(query, 1, 50) FROM system.processes
      - ![worddav4dfcf7eb68e118010709b02b0f154907.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav4dfcf7eb68e118010709b02b0f154907.png)
        - system.query_log
        - system.part_log
        - SYSTEM FLUSH LOGS 对日志表进行刷盘
      - query tracing: SET send_logs_level = 'trace'
    - 监控
      - system.events
      - system.metrics
      - system.asynchronous_metrics
      - system.processes
      - system.query_log
      - system.query_thread_log
  - [{+}](<https://presentations.clickhouse.tech/group_by/#11>)<https://presentations.clickhouse.tech/group_by/+> : 关于聚合的实现方法讨论
    - 单机多核
      - 低效的方法: 将数据读入数组, 对数组进行排序, 遍历数组 计算聚合
        - 优点: 聚合函数接口简单, 便于reduce
        - 缺点: 令N为数据总数, M为键数, 当N>M时, 内存的复杂度是O(N), 不是O(M)
      - 高效的方法: 读取数据, 将其放入关联数组, 变成 "键元组 -> 聚合函数"
        - 关联数组包括: 查找表, 哈希表, 二叉树, 跳表, B数, Tri表等
      - 分治
        - 不同线程读取不同数据, 独立聚合到本地哈希表. 读取所有数据后, 将所有哈希表合并
        - 缺点: O(M)工作是顺序执行的, 并行度很差
      - 分片
        - 原文: "

对于每个数据块，我们分两个阶段执行聚合：阶段1。不同的线程将处理该块的不同块，这些块具有时间。在每个流中，使用单独的哈希函数，我们将密钥哈希到流号中并记住它。hash：key-> bucket_num阶段2。每个流都经过整个数据块，并且仅使用具有所需存储桶编号的行进行聚合。修改：在一个阶段中一切皆有可能-然后每个线程将再次从所有字符串中计算哈希函数：如果价格便宜，这是合适的。 优点：+具有高基数和均匀分布的密钥，可以很好地扩展；+意识形态上的简单。缺点：-如果数据量在各个键之间分布不均，那么阶段2的伸缩性将不佳。这是典型的情况。几乎总是根据功率定律来分配密钥的数据量。更多缺点：-如果块大小很小，则结果是多线程的粒度太小：同步的开销很大；-如果块大小很大，则高速缓存局部性很差；-在第二阶段，将部分内存带宽乘以线程数；-您需要再计算一个哈希函数，它必须独立于哈希表中的一个；  
"

  -     -       -         - 没看懂...
      - 哈希表的并行合并
        - 在不同的流中, 将获得的哈希表大小调整为固定大小, 也就是将 一些键 合并成 键子集
        - 合并时, 对 键子集 进行合并, 合并时可以并行
        - 缺点: 代码复杂
      - 有序合并哈希表
        - 哈希表中的数据必须按哈希函数的其余部分除以哈希表的大小（直至冲突解决链）的顺序排列
        - 使用 有序迭代器 遍历并合并 多个哈希表
        - 优点: 合并是就地完成的, 适用于外部存储
        - 缺点: 代码复杂, 合并阶段未并行化, 优先队列变慢? ...
      - Robin Hood合并哈希表: [{+}](<https://programming.guide/robin-hood-hashing.html>)<https://programming.guide/robin-hood-hashing.html+>
        - 数据完全由哈希函数的其余部分除以哈希表的大小来排序 ??
        - 优点: 算法简洁, 适用于外部存储
        - 缺点: 合并阶段未并行化, 优先队列变慢?
      - 互斥下的共享哈希表
        - 优点：非常简单。缺点：可伸缩性不利
      - 不同互斥对象下的许多小哈希表
        - 使用单独的哈希函数选择要放入的那个。
        - 缺点：-通常，数据分布非常不均匀，流将在同一个热存储桶中竞争。-如果哈希表较小，则速度太慢。
        - 优点：如果数据由于某种原因而均匀分布，那么它将以某种方式扩展
      - 使用自旋锁控制共享哈希表
        - 自旋锁非常危险
        - 在典型情况下，数据高度不均匀地分布，并且流将在同一热单元上竞争
      - Lock-free控制共享哈希表
        - 无锁哈希表要么无法调整大小，要么非常复杂
        - 在典型情况下，数据分布非常不均匀，并且流将在一个热单元上竞争
      - 共享哈希表+线程本地哈希表
        - 尽量让 哈希表 不共享
        - 最后，我们将所有本地哈希表合并为一个全局哈希表
        - 优点: 完美的可扩展性, 相对简单的实现
        - 缺点: -大量查找/大量指令 -总体而言速度很慢
      - 两级哈希表
        - 在第一阶段
        - 在每个流中，我们独立地将数据放入存储不同键的num_buckets = 256个哈希表中。
        - 放入哪个（篮子编号）是由另一个哈希函数或哈希函数的单独字节确定的。我们有num_threads * num_buckets个哈希表
        - 在第二阶段
        - 我们将num_threads * num_buckets哈希表的状态合并为一个num_buckets哈希表，从而使合并跨存储桶并行化
        - 优点：+完美的可扩展性；+易于实施；+奖励：哈希表调整大小将摊销；+奖励：分区带来的好处是我们免费，这对管道的其他阶段很有用。+红利：适用于外部存储器。
        - 缺点：-具有很大的基数，在合并期间完成的工作量与第一阶段相同；-低基数，有太多单独的哈希表；-基数很小，它的工作速度比平凡的方法要慢一些；
      - 分治+两级哈希表
        - 大数据量 转化成 两级哈希表
    - 多机多核
      - 包括有几种方法的讨论, 不做解析
  - [{+}](<https://presentations.clickhouse.tech/highload_fwdays2020>)<https://presentations.clickhouse.tech/highload_fwdays2020+>: 聚合函数的执行速度调优举例
    - SELECT sum❌ FROM table
      - 对性能的分析
        - 充分使用CPU多核
        - 充分使用vector processing (CPU同一指令处理数组向量)
        - 按block处理数据, 充分使用CPU缓存
        - 在RAM中缓存 非压缩 数据
      - c++编译器为什么没有进行编译优化
        - g++ 对 loop 会进行 unwrap (将循环部分展开), 没有向量化
        - clang++ 使用了 非unwrap非向量化的方式 (但更快??)
        - 对double/float的循环, 不会进行向量化
        - 对int的循环, 会进行向量化
      - 优化方法
        - 手工进行unwrap
    - SELECT key, avg(value) FROM table GROUP BY key, key的值域为0..255
      - 优化方法
        - 手工进行unwrap, 将一个循环展开中 的 结果存储在不同的结果集 (防止由于内存对象冲突, 无法unwrap), 最后合并结果集
        - 引入批处理 (类似vector processing)
        - Variation on Bucket Sort, 没看懂
  - [{+}](<https://presentations.clickhouse.tech/highload_siberia_2018/#15>)<https://presentations.clickhouse.tech/highload_siberia_2018+>: LZ4压缩算法调优
    - memcpy 在压缩场景下慢的原因
      - memcpy属于libc, libc通常是动态链接进来的, 对memcpy的调用通过PLT (Procedure Linkage Table) 转接
      - memcpy的size参数在编译时不可预测, 不能进行inline
      - memcpy的逻辑, 一部分用来处理 不成块的内存分配
  - [{+}](<https://presentations.clickhouse.tech/highload_spb_2019/#cover>)<https://presentations.clickhouse.tech/highload_spb_2019+>: 测试数据的生成
    - 问题: 内部数据测试, 在开源后无法对外使用. 需要提供一个工具, 产生特性与生产数据相同的测试数据
    - 尝试一: 概率模型
      - 缺陷: 
        - 实现复杂
        - 列较多时, 难以找到合适的概率模型
    - 尝试二: 神经网络
      - 缺陷: 慢, 且结果无法解释
    - 尝试三: 改变压缩后的数据
      - 缺陷:
        - 数据变异的方向无法控制 -> 数据的随机度没有保障
    - 尝试四: 随机排列
      - ![worddav41f73f56681fca40a69081d9b2ee6f0f.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav41f73f56681fca40a69081d9b2ee6f0f.png)
      - 单一变换 可以生成 随机数据
      - 可以将 多个 单一变换 组合在一起
    - 尝试五: 马尔科夫模型
      - 定义 一个字符串的下一个字母是x的概率
      - 按概率生成字符串
    - 工具: clickhouse-obfuscator
      - 用于从数据产生测试数据
      - 使用算法是: 随机排列和马尔科夫模型
  - [{+}](<https://presentations.clickhouse.tech/highload_spb_2020/#11>)<https://presentations.clickhouse.tech/highload_spb_2020/+>: Clickhouse的非常规用法
    - 读取字符串并进行SQL处理, 比常用字符串处理速度快
      - ![worddav0365f9a988a2123073d2b214cf57bbce.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav0365f9a988a2123073d2b214cf57bbce.png)
    - MySQL增速?? (TODO: 测试)
      - ![worddavbc3a0437db07365cbc98b52bddff1c0c.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavbc3a0437db07365cbc98b52bddff1c0c.png)
    - 分区表元数据表
      - ![worddav78209d8d93340eeb620a50b601c558cb.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav78209d8d93340eeb620a50b601c558cb.png)
  - [{+}](<https://github.com/ClickHouse/clickhouse-presentations/blob/master/internals/en/internals.pdf>)<https://github.com/ClickHouse/clickhouse-presentations/blob/master/internals/en/internals.pdf+>
    - 索引示意图 
      - ![worddav6614b7a330e9fc1354ba95f194707780.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav6614b7a330e9fc1354ba95f194707780.png)
    - 索引必须能容纳入内存 (TODO: 需测试)
    - 点选 性能不好
  - [{+}](<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup12/power_your_data.pdf>)<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup12/power_your_data.pdf+>
    - ![worddavf091e3235b71e1d52c25c7b5175bd566.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavf091e3235b71e1d52c25c7b5175bd566.png)
    - 需验证以上问题
  - [{+}](<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup19/6.%20ClickHouse%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%20%E9%AB%98%E9%B9%8F_%E6%96%B0%E6%B5%AA.pdf>)<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup19/6.%20ClickHouse%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%20%E9%AB%98%E9%B9%8F_%E6%96%B0%E6%B5%AA.pdf+>
    - ![worddav0e260a471f561f222e65924f56c5cd15.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav0e260a471f561f222e65924f56c5cd15.png)
    - ![worddavb6c727797a26df2725ad692bd57eae52.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavb6c727797a26df2725ad692bd57eae52.png)
      - 资源限制
      - 配额限制
  - [{+}](<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup23/1_ClickHouse_on_Kubernetes.pdf>)<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup23/1_ClickHouse_on_Kubernetes.pdf+>
    - kubernetes部署方案
  - [{+}](<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup24/3.%20%E9%87%91%E6%95%B0%E6%8D%AE%E6%95%B0%E6%8D%AE%E6%9E%B6%E6%9E%84%E8%B0%83%E6%95%B4%E6%96%B9%E6%A1%88Public.pdf>)<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup24/3.%20%E9%87%91%E6%95%B0%E6%8D%AE%E6%95%B0%E6%8D%AE%E6%9E%B6%E6%9E%84%E8%B0%83%E6%95%B4%E6%96%B9%E6%A1%88Public.pdf+>
    - ![worddav7448b00b66739b2a12162fb9a4d3f85b.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav7448b00b66739b2a12162fb9a4d3f85b.png)
    - MongoDB到Clickhouse的数据映射方案
  - [{+}](<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup24/4.%20ClickHouse%E4%B8%87%E4%BA%BF%E6%95%B0%E6%8D%AE%E5%8F%8C%E4%B8%AD%E5%BF%83%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E8%B7%B5%20.pdf>)<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup24/4.%20ClickHouse%E4%B8%87%E4%BA%BF%E6%95%B0%E6%8D%AE%E5%8F%8C%E4%B8%AD%E5%BF%83%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E8%B7%B5%20.pdf+>
    - ![worddav079e952a5a84666327978cd253fba8e8.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav079e952a5a84666327978cd253fba8e8.png)
      - 横向扩展对性能的影响
      - 数据预热对性能的影响
    - ![worddav7074d5d03b9356c3773be8113d00be7f.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav7074d5d03b9356c3773be8113d00be7f.png)
      - 参数配置
  - [{+}](<https://presentations.clickhouse.tech/meetup24/5.%20Clickhouse%20query%20execution%20pipeline%20changes/#pipeline-in-clickhouse-current>)<https://presentations.clickhouse.tech/meetup24/5.%20Clickhouse%20query%20execution%20pipeline%20changes/#pipeline-in-clickhouse-current+>
    - Clickhouse的任务安排
      - ![worddav77b78466d603ef7da4003f6637cb5bcf.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav77b78466d603ef7da4003f6637cb5bcf.png)
      - 参数: experimental_use_processors (TODO: 需要测试)
    - 流水线动图
      - ![worddav3add51caa2954898a4387e9cb92c6ce9.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav3add51caa2954898a4387e9cb92c6ce9.png)
    - 参数 max_bytes_before_external_sort (TODO: 需要测试)
  - [{+}](<https://presentations.clickhouse.tech/meetup26/new_features/#11>)<https://presentations.clickhouse.tech/meetup26/new_features/#11+>
    - TTL特性
      - ![worddavcdb1ff00a69fc676a4691058da740bb3.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavcdb1ff00a69fc676a4691058da740bb3.png)
  - [{+}](<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup29/ClickHouse-MeetUp-Unusual-Applications-sd-2019-09-17.pdf>)<https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup29/ClickHouse-MeetUp-Unusual-Applications-sd-2019-09-17.pdf+>
    - 使用内存高时, 如何处理
      - 使用临时表
        - ![worddav37754434a67a475675e35e1479c7f99f.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav37754434a67a475675e35e1479c7f99f.png)
      - 使用合适的partition, 使用排序键进行过滤
        - ![worddav3d80d25cec06931340e39ee9514c7b36.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav3d80d25cec06931340e39ee9514c7b36.png)
      - join前, 将表的过滤条件手工下发
        - ![worddav007bdfb3516c7f8b001efe5baa8f99b0.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav007bdfb3516c7f8b001efe5baa8f99b0.png)
      - optimize_read_in_order: TODO: 测试参数

  

# 执行计划 explain plan 格式

格式:

  - step ( step description) (查找setStepDescription)
    - header
    - Actions:
      - Action1
      - Action2...
    - Positions: (显示Result的position)
      - position1
      - position2...

举例:   
R740-24 :) explain actions=1 select Year+1 from ontime;   
EXPLAIN actions = 1  
SELECT Year + 1  
FROM ontime   
Query id: f68dcba0-9a68-47fc-9b97-45fd46849f66   
┌─explain───────────────────────────────────────────────────────────────────┐  
│ Expression (Projection + Before ORDER BY) │  
│ Actions: INPUT : 0 -> Year UInt16 : 0 │  
│ COLUMN Const(UInt8) -> 1 UInt8 : 1 │  
│ FUNCTION plus(Year :: 0, 1 :: 1) -> plus(Year, 1) UInt32 : 2 │  
│ Positions: 2 │  
│ SettingQuotaAndLimits (Set limits and quota after reading from storage) │  
│ ReadFromStorage (MergeTree) │  
└───────────────────────────────────────────────────────────────────────────┘   
7 rows in set. Elapsed: 0.005 sec.

  - 一共3个action: 
    - 将从引擎读出的数据, 第0列输出到Year, 作为输出0
    - 常量1作为输出1
    - plus(输出0, 输出1), 作为输出2
  - 有效输出的position为输出2

步骤列表:

  - (Projection + Before ORDER BY) 这一动作是父动作Project + 子动作 Before ORDER BY 合并后的动作 (参考tryMergeExpressions函数)
  - Projection: 抽取结果列
  - Before ORDER BY: order by 相关的运算
  - Before GROUP BY: group by 相关的运算

# 执行计划 explain pipeline 格式

举例:   
R740-24 :) explain pipeline select count(Month) from ontime group by Year;   
EXPLAIN PIPELINE  
SELECT count(Month)  
FROM ontime  
GROUP BY Year   
Query id: 47391e9a-428c-47cf-8e58-10859cf38f03   
┌─explain────────────────────────────────┐  
│ (Expression) │  
│ ExpressionTransform │  
│ (Aggregating) │  
│ Resize 40 → 1 │  
│ AggregatingTransform × 40 │  
│ StrictResize 40 → 40 │  
│ (Expression) │  
│ ExpressionTransform × 40 │  
│ (SettingQuotaAndLimits) │  
│ (ReadFromStorage) │  
│ MergeTreeThread × 40 0 → 1 │  
└────────────────────────────────────────┘   
11 rows in set. Elapsed: 0.006 sec.  
  
格式: 

  - 括号内, 是执行计划步骤
  - 其他部分是调度步骤
    - MergeTreeThread × 40 0 → 1: 40个MergeTreeThread, 每个通道有0个输入, 和1个输出

  
调度步骤列表: 

  - Resize: 输入和输出 呈 扇入(n->1) 或 扇出(1->n), 输入通道与输出通道没有绑定关系 (同一输入通道的数据可以分到不同的输出通道)
  - StrictResize: 输入和输出 呈 扇入(n->1) 或 扇出(1->n), 输入通道与输出通道有绑定关系
  - PartialSorting (Sort each block for ORDER BY) : 在block内排序
  - MergeSorting (Merge sorted blocks for ORDER BY) : 对排序后的block, 进行归并
  - MergingSorted (Merge sorted streams for ORDER BY) : 如果前一步的输出涉及多个输出流(stream), 对多个流进行归并 (使用场景? )
  - CreatingSets (Create sets before main query execution) : 子查询/JOIN的准备工作
  - CreatingSet (Create set for JOIN) Join: 12136005296676117799_6141092231629190971 : JOIN生成的中间结果

# 异常现象-1

在ontime库中, 执行以下SQL:   
select count(Month) from ontime group by Year SETTINGs max_threads=10;  
  
重复执行同一SQL会比较快, 更换max_threads执行会慢, 但clickhouse客户端显示的Elapsed仍然很小: Elapsed: 0.085 sec.   
解决: 

  - 发现 clickhouse-client cpu飙高
  - 通过perf top -p {pid} 查看, 发现replxx::History::save占比99%
  - 查看procfs, 看到clickhouse-client使用到history文件, 文件大小200M
  - 猜测history算法有误. 删除history文件后, 恢复正常

# 知识框架

  - MergeTree索引结构
    - [{+}](<https://clickhouse.tech/docs/en/engines/table-engines/mergetree-family/mergetree/>)<https://clickhouse.tech/docs/en/engines/table-engines/mergetree-family/mergetree/+>
      - Primary Keys and Indexes in Queries
        - ![worddav9e526167b1b56d6d4cea10aab1646e73.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav9e526167b1b56d6d4cea10aab1646e73.png)
        - 数据以granularity为逻辑单位进行读取
        - Mark标定了granularity的边界的数据
      - TODO: 其他部分
  - 分区表
    - [{+}](<https://clickhouse.tech/docs/en/engines/table-engines/mergetree-family/custom-partitioning-key/>)<https://clickhouse.tech/docs/en/engines/table-engines/mergetree-family/custom-partitioning-key/+>
      - ![worddav686b5564025945e053ced0502fd5380e.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav686b5564025945e053ced0502fd5380e.png)
      - 分区名命名规则: 
        - 201901_1_3_1:
          - 201901 is the partition name.
          - 1 is the minimum number of the data block.
          - 3 is the maximum number of the data block.
          - 1 is the chunk level (the depth of the merge tree it is formed from).
  - Join: [{+}](<https://clickhouse.tech/docs/en/sql-reference/statements/select/join/>)<https://clickhouse.tech/docs/en/sql-reference/statements/select/join/+>
    - ASOF JOIN: 特殊的JOIN类型, 常用于带有时间序的表合并, 类似于union
    - 性能相关
      - JOIN 与 SQL处理的其他阶段 的执行顺序, 不会进行优化调整, 始终在WHERE和聚合之前进行
      - JOIN TABLE引擎可以将Join子句的结果缓存起来, 方便下次使用
      - 有时候IN比JOIN更高效
      - 对于有dimension table的情况, 可以使用external dictionary避免反复扫描右表 ?? (测试中, 性能未见提高)
  - [{+}](<https://presentations.clickhouse.tech/cern/#23>)<https://presentations.clickhouse.tech/cern/+>
    - ![worddav3c7581d4538ef37f51d3384ca68c6905.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddav3c7581d4538ef37f51d3384ca68c6905.png)
    - ![worddavf3ed695482ebec35802b2c7c74920c78.png](/assets/01KJBYDD58AJTW5NKT3GET7B89/worddavf3ed695482ebec35802b2c7c74920c78.png)

# MaterializedMySQL

  - [{+}](<https://clickhouse.tech/docs/en/engines/database-engines/materialize-mysql/>)<https://clickhouse.tech/docs/en/engines/database-engines/materialize-mysql/+>
    - 虚拟列
      - _version
      - _sign: 删除标志: -1 表示行删除
    - Data Types Support: MySQL数据类型与Clickhouse数据类型的映射
    - DDL: 不能识别的DDL, 会直接回忽略
    - DML
      - MySQL INSERT, 会转换成 INSERT with _sign=1
      - MySQL DELETE, 会转换成 INSERT with _sign=-1
      - MySQL UPDATE, 会转换成 INSERT with _sign=-1 and INSERT with _sign=1
    - SELECT
      - 如果没有指定 _version, 则转换成 FINAL 形式, 选取MAX(_version) 的行
      - 如果没有指定 _sign, 则转换成 _sign=1, 排除已删除行
    - PRIMARY KEY/INDEX, 转换成 ORDER BY
    - 受到 optimize_on_insert 影响
  - 问题
    - 集群模式下, 复制关系是怎样的
    - 如何对位
    - 如何进行故障恢复
    - 复制效率

# TODO

  - [{+}](<https://clickhouse.tech/docs/en/sql-reference/functions/array-functions/#function-empty>)<https://clickhouse.tech/docs/en/sql-reference/functions/array-functions/#function-empty+>
  - [{+}](<https://clickhouse.tech/docs/en/interfaces/third-party/proxy/>)<https://clickhouse.tech/docs/en/interfaces/third-party/proxy/+>
  - uniq v.s uniqexact
  - quota
  - [{+}](<https://clickhouse.tech/docs/en/sql-reference/functions/bitmap-functions/>)<https://clickhouse.tech/docs/en/sql-reference/functions/bitmap-functions/+>
    - ck在腾讯的应用–流存分析.docx
  - 批量插入的方法
  - 索引选择
  - 行列转换
    - Array Join
    - groupArray
  - [{+}](<http://jackpgao.github.io/2018/05/22/ClickHouse-Data-Replication/>)<http://jackpgao.github.io/2018/05/22/ClickHouse-Data-Replication/+>
    - 34页PPT掌握ClickHouse的数据复制
  - detach的database, 如果不重新attach, 重启clickhouse会失败
  - 调试入口: DB::executeQuery
  - clickhouse log level在配置文件中设置 (/etc/clickhouse-server/config.xml)
