---
note: 01KJBZ3XV1BAQAJ591V019Q47P.md
title: 20231116 - Oracle性能测试 - 使用hammerdb
indexed_at: 2026-03-05T09:17:42.501103+00:00
---

## 摘要
记录使用 HammerDB 对 Oracle 11g 进行 TPC-C 基准测试的完整配置流程，包括 Docker 容器部署、sqlplus 安装、TNS 配置及连接验证。详细记录了 TPC-C 参数配置（10 warehouse、10 VU）、schema 构建过程及遇到的 ORA-12154 连接错误。

## 关键概念
- HammerDB: 开源数据库基准测试工具，支持 TPC-C/TPC-H 等多种测试基准
- TPC-C: 事务处理性能委员会的联机事务处理 (OLTP) 基准测试标准
- Warehouse: TPC-C 测试中的数据仓库单位，代表测试规模
- Virtual User (VU): 并发虚拟用户数，模拟并发连接数
- TNSNAMES.ORA: Oracle 网络配置文件，定义数据库连接描述符

## 关联笔记
- 01KJBZ449CMKM8CQ045S94KF5E.md: 使用 hammerDB 进行 Oracle 并发压力测试的后续性能测试结果
- 01KJBZ42D6BDXBV93D0EE08ZS2.md: hammerdb 抓包诊断 Oracle SQL 的问题排查记录
- 01KJBZ3ZC6ZAF7W1ZSX2HBWT21.md: Oracle 运维技巧，包含 sqlplus 和 rman 使用方法
