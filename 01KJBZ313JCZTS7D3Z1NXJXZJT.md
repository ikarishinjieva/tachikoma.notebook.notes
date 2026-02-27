---
title: 20230829 - OB tx_id 的分配路径
confluence_page_id: 2589091
created_at: 2023-08-29T11:11:31+00:00
updated_at: 2023-08-29T11:11:31+00:00
---

产生tx_id的路径

  - ObTxDescMgr::add / add_with_txid
    - tx_id_allocator_
      - ObTransService::gen_trans_id_
        - gti_source_->get_trans_id

调用tx_desc_mgr_.add的调用点: 

  - ObTransService::start_tx
  - ObTransService::get_read_snapshot
    - ![image2023-8-29 19:7:17.png](/assets/01KJBZ313JCZTS7D3Z1NXJXZJT/image2023-8-29%2019%3A7%3A17.png)
    - 使用RR/SERIALIZATION隔离级别, 并且事务的隔离级别与配置不同, 会生成一个全局快照, 需要产生新的事务
  - ObTransService::create_global_implicit_savepoint_
  - ObTransService::create_explicit_savepoint
