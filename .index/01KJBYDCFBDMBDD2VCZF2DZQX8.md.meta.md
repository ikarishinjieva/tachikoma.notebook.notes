---
note: 01KJBYDCFBDMBDD2VCZF2DZQX8.md
title: 20210606 - 从binlog中分离SQL, 进行统计
indexed_at: 2026-03-05T07:28:13.679733+00:00
---

## 摘要
记录从 MySQL binlog 中提取 Rows_query 类型 SQL 语句的命令行流程。使用 mysqlbinlog 解析二进制日志，通过 awk 分离并合并 SQL 到同一行，最后用 pt-query-digest 分析 SQL 分布。

## 关键概念
- mysqlbinlog: MySQL 官方工具，用于解析二进制日志文件
- Rows_query: binlog 中记录实际 SQL 语句的事件类型
- pt-query-digest: Percona Toolkit 工具，用于分析 SQL 日志和统计分布
- awk 模式匹配: 使用模式范围语法提取特定日志段落

## 关联笔记
- 01KJBYF973H98W1RNZ05SS50KE.md: 记录使用 llvm-mctoll 反编译 mysqlbinlog 工具的尝试
- 01KJBYF33K90HXENWZZKHBND30.md: 分析 mysqlbinlog 反编译生成的 BC 文件和 C 代码
- 01KJBZAF5A9DS1CND4J22B9X6V.md: 提及使用 mysqlbinlog 工具查看 binlog 定位崩溃前后的操作
