---
note: 01KJBZ4FXPMAFNKRB7WKZED2CX.md
title: 20231217 - 进行oceanbase编译
indexed_at: 2026-02-27T10:41:13.528331+00:00
---

用户要求我分析这篇关于 OceanBase 编译的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. **标题**: 20231217 - 进行 oceanbase 编译

2. **内容概览**:
   - 克隆代码仓库
   - Docker 环境配置（拉取镜像、启动容器）
   - 在容器内编译 OceanBase（build.sh init, build.sh debug, make）
   - 初始化目录结构（clog, slog, ilog 等）
   - 启动 observer 进程（带大量参数）
   - 初始化数据库（bootstrap）
   - 检查数据库是否正常

3. **标签**: 应该包括 OceanBase、编译、Docker、数据库、调试等关键词

4. **摘要**: 这是一篇关于如何在 Docker 环境中编译和启动 OceanBase 数据库的操作记录

5. **关键概念**: 
   - OceanBase: 分布式数据库
   - observer: OceanBase 的核心服务进程
   - bootstrap: 数据库初始化引导过程
   - clog/slog/ilog: 各种日志文件类型
   - Docker 容器: 编译环境

6. **关联笔记**: 查看其他笔记列表，大部分是 MySQL、Clickhouse 相关的，与 OceanBase 直接关联的不多。但有一些数据库相关的：
   - 01KJBYD9S0A3VZWG60QKDTAF49.md: innodb 刷脏页 - 数据库存储引擎相关
   - 01KJBYD9TKEMSF0Z4RF5DHT1QK.md: MySQL 刷脏页研究 - 数据库存储相关
   - 01KJBYDAZJTQDJ7X3BQ52XVSSR.md: 临时磁盘表 - 数据库存储相关

   但这些主要是 MySQL 相关的，与 OceanBase 编译没有直接关联，所以可能应该写「无」或者找最接近的数据库研究类笔记。

实际上这篇笔记主要是 OceanBase 的编译和部署流程，与其他笔记的关联度确实不高。我应该生成最终的索引输出。## 标签
OceanBase, 编译, Docker, 数据库, 调试, 部署

## 摘要
记录在 Docker 容器中编译和启动 OceanBase 数据库的完整流程。包括代码克隆、镜像拉取、容器启动、编译构建、目录初始化、observer 进程启动及数据库 bootstrap 初始化。

## 关键概念
- observer: OceanBase 的核心服务进程，处理 SQL 请求和存储引擎
- bootstrap: 数据库集群初始化引导过程，指定 zone 和 server
- clog/slog/ilog: OceanBase 的各类日志文件（commit log、schema log、ilog）
- build.sh debug: OceanBase 的调试版本编译脚本
- resource pool: 通过 memory_limit、cpu_count 等参数配置资源限制

## 关联笔记
无
