---
title: 20210622 - Clickhouse distributed join 学习
confluence_page_id: 1147036
created_at: 2021-06-22T06:07:26+00:00
updated_at: 2021-06-22T10:08:26+00:00
---

# TODO

  - 配置: distributed_product_mode
  - 与max_parallel_replicas的联动
  - sample by的测试, intHash64有效, 但Int64无效

# 实验

## distribute join distribute

select * from all_tt t1 join all_aa t2 using a

![image2021-6-22 13:18:20.png](/assets/01KJBYDDA1R8E2ZZK7ERJ8BKKM/image2021-6-22%2013%3A18%3A20.png)

对于每个节点: 先准备右表 (分布式), 再与本地表Join, 最后进行分布式聚合

右表查询进行N次

每个节点都需要执行: select * from local_tt t1 join all_aa t2 using a

## distribute global join distribute

select * from all_tt t1 global join all_aa t2 using a

![image2021-6-22 13:31:35.png](/assets/01KJBYDDA1R8E2ZZK7ERJ8BKKM/image2021-6-22%2013%3A31%3A35.png)

右表的分布式子查询先执行

对于每个节点: 将右表执行结果存入Memory, 之后再与本地表Join, 最后进行分布式聚合

右表查询只进行一次, 再初始节点上进行, 再分发给各个节点

每个节点都需要执行: select * from local_tt t1 join MEMORY t2 using a

# 运维建议

  - <https://clickhouse.tech/docs/en/sql-reference/operators/in/#select-distributed-subqueries>
    - 对于GLOBAL JOIN的运维建议: 
      - 右表(临时表) 不进行数据排重, 可以手工增加DISTINCT, 减少数据传输量
      - 右表(临时表) 会发送到所有节点, 不会考虑节点的网络拓扑, 会引起网络流量高, 且不会受节制. 尽量调整网络拓扑, 部署在同一数据中心, 使得流量成本降低
      - 尽量调整数据分布, 避开GLOBAL IN
