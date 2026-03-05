---
note: 01KJBZ313JCZTS7D3Z1NXJXZJT.md
title: 20230829 - OB tx_id 的分配路径
indexed_at: 2026-03-05T08:58:35.747167+00:00
---

## 摘要
记录 OceanBase 数据库中 tx_id 的生成路径，从 ObTxDescMgr::add 到 gti_source_->get_trans_id 的完整调用链。列出四个调用 tx_desc_mgr_.add 的入口点，包括事务启动、读快照获取和保存点创建场景。

## 关键概念
- ObTxDescMgr: 事务描述符管理器，负责分配和管理 tx_id
- ObTransService: 事务服务核心类，提供事务 ID 生成和事务启动功能
- tx_id_allocator: 事务 ID 分配器，通过 GTI 服务获取全局事务 ID
- 全局快照: RR/SERIALIZATION 隔离级别下生成新事务时创建的快照

## 关联笔记
- 01KJBZ2TD63SRBNM9GPKBXR5G4.md: 同样涉及 ObTransService::gen_trans_id 和 tx_id 生成机制，描述 GTI RPC 申请 ID 的缓存流程
