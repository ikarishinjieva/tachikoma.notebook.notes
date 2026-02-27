---
title: 20221021 - clickhouse mutations运行代码分析
confluence_page_id: 1934337
created_at: 2022-10-21T03:55:58+00:00
updated_at: 2022-10-21T15:08:34+00:00
---

版本: 20.1.8.41

ReplicatedMergeTreeQueue::updateMutations

  - 从zk中取出mutations
  - 分发到 内存结构 mutations_by_znode 和 mutations_by_partition
  - 由 ReplicatedMergeTreeQueue::pullLogsToQueue 触发

StorageReplicatedMergeTree::mergeSelectingTask

  - 根据mutation 生成 log entry
  - TODO: 是否能限制住mutation合并的数量

StorageReplicatedMergeTree::queueTask

  - 从queue中获取 log entry
  - StorageReplicatedMergeTree::executeLogEntry
    - 如果 log entry 类型为 LogEntry::MUTATE_PART, 进行 mutation 处理
      - tryExecutePartMutation
        - queue.getMutationCommands: commands 的获取: 
          - 从 mutations_by_partition 中, 按照当前version和 (log entry中的version) 差距, 找到一段 mutations, 放入commands

测试: 

  - prefer_fetch_merged_part_time_threshold
  - prefer_fetch_merged_part_size_threshold
