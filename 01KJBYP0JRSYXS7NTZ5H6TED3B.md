---
title: 20221008 - OB文档阅读
confluence_page_id: 1933997
created_at: 2022-10-08T07:29:51+00:00
updated_at: 2022-10-24T08:20:48+00:00
---

# 问题1

<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000367358>

![image2022-10-8 15:20:46.png](/assets/01KJBYP0JRSYXS7NTZ5H6TED3B/image2022-10-8%2015%3A20%3A46.png)e

# 问题2

<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000367357>

![image2022-10-8 15:22:14.png](/assets/01KJBYP0JRSYXS7NTZ5H6TED3B/image2022-10-8%2015%3A22%3A14.png)

# 问题3

<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000367369>

![image2022-10-8 15:29:43.png](/assets/01KJBYP0JRSYXS7NTZ5H6TED3B/image2022-10-8%2015%3A29%3A43.png)

# 问题4

<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000354985>

![image2022-10-10 14:37:55.png](/assets/01KJBYP0JRSYXS7NTZ5H6TED3B/image2022-10-10%2014%3A37%3A55.png)

  - NUMA差异? 
  - Cstate/Pstate值得借鉴

# 问题5

<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000354647>

![image2022-10-10 17:22:32.png](/assets/01KJBYP0JRSYXS7NTZ5H6TED3B/image2022-10-10%2017%3A22%3A32.png)

统一为 OBProxy更容易理解

# 问题6

<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355640>

![image2022-10-11 9:29:3.png](/assets/01KJBYP0JRSYXS7NTZ5H6TED3B/image2022-10-11%209%3A29%3A3.png)

为什么要挑出 合并中的副本, 合并会引起明显的阻塞? 

# 问题7

<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355578>

![image2022-10-11 13:59:59.png](/assets/01KJBYP0JRSYXS7NTZ5H6TED3B/image2022-10-11%2013%3A59%3A59.png)

# 问题8

<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355895>

![image2022-10-14 15:20:5.png](/assets/01KJBYP0JRSYXS7NTZ5H6TED3B/image2022-10-14%2015%3A20%3A5.png)

# 问题9

<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355046>

![image2022-10-15 21:3:9.png](/assets/01KJBYP0JRSYXS7NTZ5H6TED3B/image2022-10-15%2021%3A3%3A9.png)

这两个规则的意义? 

# 问题10

<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355007>

![image2022-10-16 20:31:23.png](/assets/01KJBYP0JRSYXS7NTZ5H6TED3B/image2022-10-16%2020%3A31%3A23.png)

# 问题11

<https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000579678>

"产品形态" 里 包括了命令行参数列表, 比较难理解. 建议标题直接就改成 "命令行参数"

# TODO

  - ob_route_policy 工作机制, 为什么在observer层: <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355796>
  - follower/.../ 的部署架构咨询
  - 数据合并的相关
    - 错峰合并? : <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355797>
      - Zone合并, observer合并
    - 监控/影响/故障处理/日志
  - resource unit等 资源限制的关系
  - isolate zone和stop zone的区别: <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355933>
    - 集群部分故障后的流量情况和应急措施
  - 数据如何均衡
  - OB内部表的命名规范? : __all_server
  - 常用参数表: <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355581>
    - 目标1: 各参数配置不合理时的表现和应对 (故障处理手册)
    - 目标2: 将各参数归纳到某个目标地: 检查单/机制培训/故障处理手册
  - 转储: 
    - 自动转储: <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355955>
      - memstore在转储时的内存使用, 是否会突破限制
      - 问题: 转储的状态表名为: merge_info? : <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355957>
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355958>
      - 转储参数不合适 时的系统表现/运维经验
  - 合并: 
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355959>
      - major compaction的状态名: __all_zone.merge_status
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355961>
      - 定时合并 超时后的处理
  - 内存: 
    - 内存的统计机制? <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355603>
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355603>
      - KV-cache的淘汰机制的影响
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355605>  

      - 查看内存的使用量, 是否能排查某个占用内存过大的Query
      - 各内存统计表 的命名规则
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355606>
      - plan cache 命中率的监控 和 后处理
  - 线程: 
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355611>
      - 什么是PX SQL (parallel SQL)
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355612>
      - obstack的使用
      - 大小查询 隔离使用资源的机制 的原理
      - 线程名 从OS是否能查看
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355613>
      - 后台线程 数量不足时的表现 和 后处理
  - 索引
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000357177>
      - 局部索引 的副本, 与 主数据 相同?
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000357178>
      - 全局索引 的数据位置?
  - Locality
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355097>
      - 修改Locality后, 数据调整的进度和错误处理
  - Paxos
    - 如何 重启一个分区的Paxos协调? 
  - 性能
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000357019>
      - 各队列 排队状况的意义
  - OCP
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000357029>
      - SQLTuningAdvisor 的使用
  - 参数
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355088>
      - 加入规范
  - SQL诊断
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355070>
      - 供参考, 建立业务模型诊断流程
  - 日志传输服务: 
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000367350>
      - 何时备集群会主动拉取
  - 性能测试
    - 各个测试中, 都有推荐参数列表, 需要理清差异
  - Restore Point
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355059>
      - 恢复点的机制? 是否会 顶死 undo log
  - 备份
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355893>
      - 自动备份/存储空间检查/自动备份清理, 是由OCP触发, 还是由OB触发
      - 日志分片 功能 的 目标场景? 
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355897>
      - 停止备份后的资源收尾?
  - 数据校验
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355913>
      - 校验的正确性 与 集群信息 是什么关系
      - 校验算法? 
  - 恢复
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355918>
      - restore_concurrency
      - _restore_idle_time
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355921>  

      - PREVIEW?
  - 主备
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355044>
      - 备集群不能主动触发大版本冻结 的原因? 

      - "自动执行重建操作" 的条件
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355050>  

      - enable_monotonic_weak_read 单调读 的机制? 
  - 加密
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000354763>
      - 存储加密: SM4 实践
      - "internal 方式的透明表空间加密" ??
  - 告警
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355557>
      - 告警项 里有一些无法理解的部分
  - 扩缩容
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355566>
      - 超配 的使用?
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355567>
      - 扩缩容的过程中, 查看平衡进度
  - 巡检
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355024>
      - 合并操作的时长? 
  - 排错  

    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355008>
      - addr2line 用于转换堆栈
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355013>
      - 迁移缓慢 相关的参数
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355014>
      - 写入的速度大于转储的速度，导致 MEMstore 耗尽 的排查细节

  - 数据迁移
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355327>
      - 服务端限流阈值
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000354522>
      - 校验集?
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355301>
      - 分区表的推荐条件: 单表行数可能超过 10 亿行或者单表容量超过 2000GB，推荐进行创建分区表。
    - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355302>
      - 调优经验, 需要加到已有文档中
      - "对 partition 进行运维时，请注意 drop 或 truncate 操作会导致全局索引失效。"
  - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000354596>
    - 布隆过滤器 的使用? 
  - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355154>
    - [resource pool: 每个资源池 只有一个资源单位的意义? 资源池到底装了什么?   
](<https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000355154>)

# 文档TODO

  - <https://www.oceanbase.com/docs/enterprise-oma-doc-cn-10000000000579675>
  - <https://www.oceanbase.com/docs/enterprise-oceanbase-database-cn-10000000000354584>
