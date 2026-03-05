---
note: 01KJBZ3KF8JB5YRT07J6JXC86C.md
title: 20231031 - 诊断OMS界面流程
indexed_at: 2026-03-05T09:09:32.078630+00:00
---

## 摘要
记录 OMS 界面启动 Store 操作提示成功但实际未生效的问题排查过程。通过操作编号 3000001 在 dss-operation.log 和 oms-web.log 中追踪请求执行链路，分析从界面调用到后台任务执行的完整流程。

## 关键概念
- dss-operation.log: OMS 后台任务执行日志，记录 request/node/order/step 的执行流程
- oms-web.log: OMS Web 服务日志，记录 API 请求响应和 trace ID
- STORE_RESTART_TASK: Store 重启任务类型，包含 CLEAR_CRAWLER 和 RESTORE_CRAWLER 两个步骤
- requestId/traceId: 用于跨日志文件追踪同一请求的唯一标识符

## 关联笔记
- 01KJBZ3FQT8FKX47BF6Q8C4X7J.md: 同日期 OMS 日志系统分析笔记，包含 oms-web.log 和 dss-operation.log 的详细分析方法
- 01KJBZ4E6HJKFS7FGETVFA76RD.md: OMS 全量验证链路整理，涉及 OMS 组件和日志体系
