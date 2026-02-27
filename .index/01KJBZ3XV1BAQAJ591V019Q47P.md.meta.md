---
note: 01KJBZ3XV1BAQAJ591V019Q47P.md
title: 20231116 - Oracle性能测试 - 使用hammerdb
indexed_at: 2026-02-27T10:38:35.892479+00:00
---

用户要求我分析这篇关于 Oracle 性能测试的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **笔记主题**：使用 hammerdb 进行 Oracle 性能测试
2. **核心内容**：
   - 启动 hammerdb 容器
   - 安装 Oracle sqlplus
   - 配置 tnsnames.ora
   - 测试数据库连接
   - hammerdb 配置（TPC-C 基准测试）
   - 数据加载（buildschema）
   - 遇到的问题（ORA-12154 错误）

3. **标签**：Oracle, hammerdb, TPC-C, 性能测试, 数据库基准测试

4. **摘要**：记录使用 hammerdb 对 Oracle 数据库进行 TPC-C 性能测试的过程，包括环境配置、连接测试和 schema 创建。测试中遇到 TNS 连接标识符解析错误。

5. **关键概念**：
   - hammerdb: 开源数据库性能测试工具
   - TPC-C: 在线事务处理基准测试标准
   - tnsnames.ora: Oracle 网络配置文件
   - Warehouse: TPC-C 测试中的数据仓库单位

6. **关联笔记**：查看其他笔记列表，主要是 MySQL、Clickhouse、性能分析相关内容。这篇是 Oracle 性能测试，与 MySQL 刷盘研究、performance_schema 使用等可能有一定关联（都是数据库性能相关），但关联度不高。

最相关的是 performance_schema 的日常使用和 MySQL 刷盘研究，因为它们都涉及数据库性能分析。不过这些笔记的时间戳是 2021 年，而当前笔记是 2023 年 11 月，时间上有一定差距。

根据内容判断，这篇主要聚焦 Oracle 和 hammerdb，与其他笔记的直接关联有限。可以选择列出性能分析相关的笔记，或者标注为无直接关联。## 标签
Oracle, hammerdb, TPC-C, 性能测试, 数据库基准测试, 容器

## 摘要
记录使用 hammerdb 对 Oracle 数据库进行 TPC-C 性能测试的完整流程，包括容器启动、sqlplus 安装、TNS 配置和连接测试。测试过程中配置了 10 个 Warehouse 和 10 个虚拟用户，但在创建 schema 时遇到 ORA-12154 连接错误。

## 关键概念
- hammerdb: 开源数据库性能基准测试工具，支持 TPC-C/TPC-H 等标准测试
- TPC-C: 在线事务处理 (OLTP) 性能基准测试标准，以 Warehouse 数量衡量规模
- tnsnames.ora: Oracle 客户端网络配置文件，定义数据库连接标识符
- Warehouse: TPC-C 测试中的数据量单位，代表一套完整的业务数据

## 关联笔记
- 01KJBYD9T7VWC3J9T1JMVWGWMB.md: 同属数据库性能测试相关，涉及 performance_schema 使用
- 01KJBYD9VE4NJQXE9P0VKM16Q4.md: 同为数据库性能问题研究，涉及刷盘性能分析
