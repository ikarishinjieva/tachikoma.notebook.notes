---
title: 20210404 - MySQL刷脏页研究
confluence_page_id: 753795
created_at: 2021-04-04T11:38:40+00:00
updated_at: 2022-05-27T12:51:00+00:00
---

# 参考文章1

<https://www.percona.com/blog/2019/12/18/give-love-to-your-ssds-reduce-innodb_io_capacity_max/>

  - SSD 如果部分数据是固定数据不变更, 频繁写入的范围变小, 导致这部分的寿命变低
  - 因此: innodb_io_capability_max 不是越大越好, 细水长流容易保住磁盘的寿命

# 参考文章2

<https://www.percona.com/blog/2020/01/22/innodb-flushing-in-action-for-percona-server-for-mysql/>

  - InnoDB 刷脏分为三种原因: 
    - 根据 buffer pool 脏页比例刷盘
    - Free list 不足, 引起刷盘
    - 由于redo log 空闲不足, 引起刷盘 -> 以redo log未checkpoint的页长度, 评估应该如何刷脏

  - 根据 buffer pool 脏页比例刷盘
    - 相关参数: 
      - innodb_max_dirty_pages_pct_lwm = 10
      - innodb_max_dirty_pages_pct = 75
      - innodb_io_capacity = 200
    - 行为
      - 如果innodb_max_dirty_pages_pct_lwm=0, 脏页不进行预刷, 达到innodb_max_dirty_pages_pct后, 直接全力刷脏
      - 如果innodb_max_dirty_pages_pct_lwm>0, 脏页比例超过innodb_max_dirty_pages_pct_lwm开始刷脏, 刷脏页数: 
        - ![image2021-4-4 18:56:42.png](/assets/01KJBYD9TKEMSF0Z4RF5DHT1QK/image2021-4-4%2018%3A56%3A42.png)
        - 刷脏页数 = (脏页比例 / innodb_max_dirty_pages_pct) * innodb_io_capacity , 最大值为 innodb_io_capacity_max

  - Adaptive Flushing
    - 以redo log未checkpoint的页长度, 评估应该如何刷脏
    - 算法的目的: 使得Redo log的 TAIL(刷盘)的速度 赶上 HEAD(事务生成)的速度一致
    - 定义: redo log的最大容量(max checkpoint age) = (redo log各个文件大小的总和 - 文件头) * 90%
    - 相关参数: 
      - innodb_adaptive_flushing (default ON)

      - innodb_adaptive_flushing_lwm (default 10)

      - innodb_flush_sync (default ON)

      - innodb_flushing_avg_loops (default 30)

      - innodb_io_capacity (default value of 200)

      - innodb_io_capacity_max (default value of at least 2000)

    - 行为: 
      - 当LRU的脏页比例(checkpoint age) 大于 innodb_adaptive_flushing_lwm 时, 开始触发刷盘
      - ![image](https://www.percona.com/blog/wp-content/uploads/2020/01/finalPagesToFlush.png)
      - 由以下三个因素综合形成: 
        - 由 redo log age 计算的刷盘 (面对现状) (pctOfIoCapToUse * ioCap)
        - 过去一段时间, 平均的刷盘页数 (过去的能力) (avgPagesFlushed)
        - 由 过去一段时间生成的页数速率, 推导出的刷盘次数 (面对未来的增长) (pagesForLsnRate)
      - 过去的能力
        - 平均刷盘页数: avg_page_rate: avgPagesFlushed
      - 面对未来的增长
        - 平均redo log生成页数: lsn_avg_rate: avgLsnRate
        - pagesForLsnRate = 当生成页数速率保持 avgLsnRate 时, 当前应刷盘的次数 (还需经过一些封顶计算)
      - 面对现状
        - 刷盘IO能力比例: 
          - ![image](https://www.percona.com/blog/wp-content/uploads/2020/01/legacy_equation.png)
        - 曲线 (如线Legacy): 
          - ![image](https://www.percona.com/blog/wp-content/uploads/2020/01/FlushingPressure.png)
      - 最大值为 innodb_io_capacity_max
      - 削峰平谷: 
        - 对于avg_page_rate (过去的能力) 和 lsn_avg_rate (未来的需求), 每 innodb_flushing_avg_loops 轮, 或每 innodb_flushing_avg_loops 秒, 计算一次平均值

  - 为了保障Free List有足够的空闲页, 发起的刷盘
    - 相关参数:
      - innodb_lru_scan_depth
    - 行为
      - 异步刷盘, 使得 Free list 长度保持在 innodb_lru_scan_depth 之上

  - 当开启 innodb_flush_sync, 由于 redo log checkpoint 产生的刷脏, IO可超过 innodb_io_capacity_max 限制
  - 三种触发原因的关系: 
    - 脏页比例刷盘 要计算所有buffer pool instance的统计值; Adaptive flushing 要计算redo log的modified age, 所以两者必须在全局只有一个协调者, 不能分散到各个buffer pool instance中进行计算.   
因此: 这两个因素的计算 需要在 page cleaner coordinator进行, 然后下发任务给 page cleaner worker执行
    - 当worker有任务需要执行时, 会顺带计算 Free List的空闲页 数量: 
      - 如果需要刷脏, 会先根据 空闲页 的指标刷脏
      - 完成后, 在根据其他两个指标刷脏
  - 对以下报错日志的解析:   
  

```
[Note] InnoDB: page_cleaner: 1000ms intended loop took 4013ms. The settings might not be optimal. (flushed=1438 and evicted=0, during the time.)
``` 

    - 单次刷盘时间超过3000ms时触发日志
    - 针对报错的解决方案: 
      - 刷盘一页可能导致两个IO操作, 使得IOPS大于预期
      - 其他读IO的影响
      - 设备写延迟突然增大, 比如SSD发生GC
      - 对 doublewrite buffer 发生了竞争
      - page cleaner线程数量不足, 无法发挥设备IO效能

# 参考文章3

<https://www.percona.com/blog/2011/04/04/innodb-flushing-theory-and-solutions/>

# TODO

  1. 对下列日志进行处理和教学: 

```
[Note] InnoDB: page_cleaner: 1000ms intended loop took 4013ms. The settings might not be optimal. (flushed=1438 and evicted=0, during the time.)
```
