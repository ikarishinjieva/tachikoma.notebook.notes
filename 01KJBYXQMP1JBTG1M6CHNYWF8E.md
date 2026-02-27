---
title: 20221210 - 光大CK 调大后台线程数导致内存急速增长
confluence_page_id: 2130357
created_at: 2022-12-10T15:48:26+00:00
updated_at: 2022-12-10T15:48:26+00:00
---

# 前继

[20221123 - 光大 AST is too big 报错后续]

# 现象

调大后台线程数(background_pool_size) 后 (16->32), 内存极速上涨

# 日志

[clickhouse-server.tar.gz](/assets/01KJBYXQMP1JBTG1M6CHNYWF8E/clickhouse-server.tar.gz)

截取一段 CK启动的片段: 

```
//启动CK
2022.12.09 18:31:47.065461 [ 1 ] {} <Information> : Starting ClickHouse 20.1.8.41 with revision 54431
2022.12.09 18:31:47.065523 [ 1 ] {} <Information> Application: starting up
2022.12.09 18:31:47.069631 [ 1 ] {} <Debug> Application: rlimit on number of file descriptors is 65535
2022.12.09 18:31:47.069640 [ 1 ] {} <Debug> Application: Initializing DateLUT.
2022.12.09 18:31:47.069646 [ 1 ] {} <Trace> Application: Initialized DateLUT with time zone 'Asia/Shanghai'.
2022.12.09 18:31:47.069980 [ 1 ] {} <Debug> Application: Configuration parameter 'interserver_http_host' doesn't exist or exists and empty. Will use 'nf52804948' as replica host.

//读取元数据
2022.12.09 18:31:47.073627 [ 1 ] {} <Information> Application: Loading metadata from /data/stix/clickhousedata/
 
//对元数据进行初始化, 将展开对这段进行分析
...
 
//读取元数据完成
2022.12.09 18:35:43.817813 [ 1 ] {} <Debug> Application: Loaded metadata.
...
 
//CK启动完成
2022.12.09 18:35:43.818861 [ 1 ] {} <Information> Application: Listening for http://0.0.0.0:8123
2022.12.09 18:35:43.818892 [ 1 ] {} <Information> Application: Listening for connections with native protocol (tcp): 0.0.0.0:24902
2022.12.09 18:35:43.818920 [ 1 ] {} <Information> Application: Listening for replica communication (interserver): http://0.0.0.0:9009
2022.12.09 18:35:43.896809 [ 1 ] {} <Information> Application: Listening for MySQL compatibility protocol: 0.0.0.0:24903
2022.12.09 18:35:43.908340 [ 1 ] {} <Information> Application: Available RAM: 250.80 GiB; physical cores: 24; logical cores: 48.
2022.12.09 18:35:43.908365 [ 1 ] {} <Information> Application: Ready for connections.
``` 

对元数据初始化时段 进行分析:

[ck.log](/assets/01KJBYXQMP1JBTG1M6CHNYWF8E/ck.log)

排查18:35中, 所有线程中, 分配内存超过1 GiB的线程内存用量: 

```
cat /Users/tachikoma/Downloads/ck.log  | grep 'Current memory usage:' | grep '18:35' | grep GiB | sort -k4 -n 
``` 

可以得到以下线程: 

  - 线程10: 20.17 GiB

  - 线程12: 20.17 GiB

  - 线程24: 20.18 GiB

  - 线程26: 20.21 GiB

  - 线程27: 20.19 GiB

  - 线程28: 20.20 GiB

  - 线程33: 20.20 GiB

  - 线程35: 20.17 GiB

  - 线程80: 27.00 GiB

  - 这几个重点线程 共消耗 187GiB左右的内存

分析各线程当时的任务: 

  - 线程10: default.CK_BLACK_FILE_LOCAL: Executing log entry to mutate part 4f65688afb3794b6d51e09142c1191bb_92425_92433_2 to 4f65688afb3794b6d51e09142c1191bb_92425_92433_2_107342
  - 线程12: default.CK_BLACK_FILE_LOCAL: Executing log entry to mutate part 4f65688afb3794b6d51e09142c1191bb_92436_92436_0 to 4f65688afb3794b6d51e09142c1191bb_92436_92436_0_107342
  - 线程24: default.CK_BLACK_FILE_LOCAL: Executing log entry to mutate part 4f65688afb3794b6d51e09142c1191bb_92414_92414_0 to 4f65688afb3794b6d51e09142c1191bb_92414_92414_0_107342
  - 线程26: default.CK_BLACK_FILE_LOCAL: Executing log entry to mutate part 4f65688afb3794b6d51e09142c1191bb_92370_92370_0 to 4f65688afb3794b6d51e09142c1191bb_92370_92370_0_107342
  - 线程27: default.CK_BLACK_FILE_LOCAL: Executing log entry to mutate part 4f65688afb3794b6d51e09142c1191bb_92403_92411_2 to 4f65688afb3794b6d51e09142c1191bb_92403_92411_2_107342
  - 线程28: default.CK_BLACK_FILE_LOCAL: Executing log entry to mutate part 4f65688afb3794b6d51e09142c1191bb_92381_92389_2 to 4f65688afb3794b6d51e09142c1191bb_92381_92389_2_107342
  - 线程33: default.CK_BLACK_FILE_LOCAL: Executing log entry to mutate part 4f65688afb3794b6d51e09142c1191bb_92392_92392_0 to 4f65688afb3794b6d51e09142c1191bb_92392_92392_0_107342
  - 线程35: default.CK_BLACK_FILE_LOCAL: Executing log entry to mutate part 4f65688afb3794b6d51e09142c1191bb_92445_92453_2 to 4f65688afb3794b6d51e09142c1191bb_92445_92453_2_107342
  - 线程80: default.CK_BLACK_DOMAIN_LOCAL (ReplicatedMergeTreeQueue): Loading 14187 mutation entries: 0000006012 - 0000020198
    - 线程80在18:35:43执行完成, 消耗27GiB内存
    - 2022.12.09 18:35:41.601043 [ 80 ] {}  MemoryTracker: Current memory usage: 27.00 GiB.

    - 2022.12.09 18:35:43.816375 [ 80 ] {}  default.CK_BLACK_DOMAIN_LOCAL (ReplicatedMergeTreeRestartingThread): Execution took 236275 ms

结论: 

  - 积压的mutation中, 对 CK_BLACK_FILE_LOCAL 表的alter, 需要大量内存支撑. 当调大后台线程数时, mutation回放的并发度增加, 导致对内存的需求增加
  - 运维建议: 
    - background_pool_size 需要选择合适的值: 即要有足够的后台线程让mutation线程得以运行(见之前的mutation无法回放的问题分析), 又不能让mutation的并发过大, 消耗内存
    - 将 CK_BLACK_FILE_LOCAL 表的 prefer_fetch_merged_part_size_threshold和prefer_fetch_merged_part_time_threshold调整为1, 使得跳过mutation回放, 直接从另一节点上拉取part数据, 这样可能避免mutation回放带来的内存消耗
