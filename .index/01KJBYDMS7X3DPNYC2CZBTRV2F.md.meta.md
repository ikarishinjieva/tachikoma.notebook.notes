---
note: 01KJBYDMS7X3DPNYC2CZBTRV2F.md
title: 20210628 - Adaptive hash index 代码解析
indexed_at: 2026-03-05T07:32:58.193401+00:00
---

## 摘要
分析 InnoDB 自适应哈希索引 (AHI) 的数据结构、函数流程和启用条件。涵盖查找函数 btr_search_guess_on_hash 和更新函数 btr_search_info_update 的调用链及判断条件。

## 关键概念
- btr_search_sys: 全局 AHI 系统结构，包含分区锁和哈希表数组
- hash fold: 检索条件签名折叠后的哈希值，用于定位哈希表中的记录地址
- buf_chunk_map_reg: 全局映射表，将记录地址 (rec_t) 二次映射到 buffer pool 的 block
- latch_mode: B-Tree 搜索时的锁模式，决定是否可以启用 AHI 查找

## 关联笔记
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 同一代码文件 btr0sea.cc 的调用栈出现在刷盘慢导致卡顿的堆栈分析中
