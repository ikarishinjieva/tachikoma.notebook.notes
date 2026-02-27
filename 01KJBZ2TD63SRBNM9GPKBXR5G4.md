---
title: 20230810 - OB如何增加GTID
confluence_page_id: 2589032
created_at: 2023-08-11T06:15:24+00:00
updated_at: 2023-08-15T16:04:41+00:00
---

查找ObLogBR (OB从redo log中订阅的 日志项) 中, 可能存在的唯一标号

  - 关于日志项中 Checkpoint的来源
    - ObLogInstance::next_record (由oblogproxy调用)
      - (*record)->setCheckpoint(last_heartbeat_timestamp)
        - HEARTBEAT record会更新 心跳时间戳

  - LobBR构造函数: 

```
int ObLogBR::init_data(const RecordType type,
    const uint64_t cluster_id,
    const int64_t tenant_id,
    const uint64_t row_index,
    const common::ObString &trace_id,
    const common::ObString &trace_info,
    const common::ObString &unique_id,
    const int64_t schema_version,
    const int64_t commit_version,
    const int64_t part_trans_task_count,
    const common::ObString *major_version_str)
``` 
  -     - 其中需要调查: 
      - unique_id

```
ObLogFormatter::init_binlog_record_for_dml_stmt_task_
{
	init_dml_unique_id_( ..., &dml_unique_id)
	br->init_data(..., dml_unique_id)
}
 
ObLogFormatter::init_dml_unique_id_
{
	row_no = stmt_task.get_row_index()
	log_entry_task.get_log_lsn(redo_log_lsn)
	dml_stmt_unique_id(part_trans_info_str, redo_log_lsn, row_no)
	dml_stmt_unique_id.customized_to_string(...)
}
 
其中: 
	- part_trans_info_str 的初始化在 PartTransTask::to_string_part_trans_info_
		- part_trans_info_str = ("%s"DELIMITER_STR"%ld", tls_str_, trans_id_.get_id())
``` 

        - tls_str_: ObLogLSFetchMgr::init_tls_info_: 
          - 格式: "%lu_%ld", tls_id.get_tenant_id(), tls_id.get_ls_id().id()
          - tls_id: Tenant Log service ID 由两部分构成: TenantID, LogServiceID. 对应于OB的日志流: <https://www.oceanbase.com/docs/community-observer-cn-10000000000901315>
            - LogServiceID从 元数据表中获取: "SELECT LS_ID FROM CDB_OB_LS WHERE LS_ID != 1 AND TENANT_ID = "

        - trans_id_.get_id

          - PartTransDispatcher::alloc_task: 
            - ObCDCPartTransResolver::obtain_task_ (tx_id)
              - ObTransService::gen_trans_id 生成transaction ID, 通过GTI rpc申请一批ID放在cache, 
                - GTI的服务端: ObTransIDService::handle_request
                  - 调用: ObIDService::get_number
    - commit_version: 时间戳
      - ObCDCPartTransResolver::handle_commit_
        - trans_commit_version = get_trans_commit_version_(submit_ts, commit_log.get_commit_version().get_val_for_logservice())
        - commit_log.commit_version 是SCN ??
          - ObPartTransCtx::submit_commit_log_
            - log_commit_version = ctx_tx_data_.get_commit_version()
            - ObTxCommitLog commit_log(log_commit_version, ...)
              - ObPartTransCtx::generate_commit_version_()
                - ctx_tx_data_.set_commit_version(gts, ...)
                  - ObTransService::get_gts_ 从GTS获取时间戳

  - 订阅日志, 是通过LSN: 

```
int FetchLogARpc::async_fetch_log(
    const palf::LSN &req_start_lsn,
    const int64_t client_progress,
    const int64_t upper_limit,
    bool &rpc_send_succeed)
```

  - SCN跟时间戳的关系: calc_scn_to_timestamp_expr

  - LSN和SCN的关系: <https://www.oceanbase.com/docs/common-oceanbase-database-cn-10000000001577139>
    - OB_LOG_STAT表中, 存在begin_lsn和begin_scn两列, 赋值函数为: ObAllVirtualPalfStat::insert_log_stat_
      - palf_stat.begin_lsn_: 来源PalfHandleImpl::stat: log_engine_.get_min_block_info(min_block_id, min_block_min_scn)
        - begin_lsn_ = LSN(min_block_id * PALF_BLOCK_SIZE)
      - palf_stat.begin_scn_: 来源PalfHandleImpl::stat: log_engine_.get_min_block_info(min_block_id, min_block_min_scn)
        - begin_scn = min_block_min_scn
      - 重点是: log_engine_.get_min_block_info(min_block_id, min_block_min_scn) 输出的两个值
        - begin_lsn: min_block_id是从 LogBlockMgr::get_block_id_range 获取, 获取了Log系统中的最小块号
        - begin_scn: 从get_block_min_scn(min_block_id) 获取, 从最小的块中的header读取min SCN

  - 备份恢复后, 使用什么作为日志传输的起始点位  

    - 在Recover过程中, 处理Log订阅部分: ObRecoveryLSService::do_work
      - ObRecoveryLSService::seek_log_iterator_: 通过SCN查找LSN

# 结论:

  - 三个ID: LSN, SCN, TX_ID
    - ObRecoveryLSService::seek_log_iterator_: 通过SCN二分查找LSN

    - 我们可以仿照: 使用TX_ID二分查找LSN, (前提是TX_ID在clog中是顺序的)
  - 如果上述仿照是可实现的, 那么: 可以使用TX_ID来生成GTID
    - 另一个问题: TX_ID不仅是业务事务使用, 内部事务也会使用, 所以内部事务的GTID可以用空事务代替, 使得GTID连续
