---
title: 20210428 - druid连接池 连接维护逻辑整理
confluence_page_id: 754031
created_at: 2021-04-28T04:55:19+00:00
updated_at: 2021-04-29T08:45:35+00:00
---

# 整理

  - 版本: 7f0956b28c2f82cf5ffa901ad07dc54de96104d8
  - 创建连接的线程: CreatorThread
    - 若开启了keepalive, 并且总连接数低于minIdle, 建立新连接
    - 若有线程在等待连接, 建立新连接
    - 连接数超过maxActive, 不允许建立新连接
  - 关闭连接的线程: DestroyThread: 每 timeBetweenEvictionRunsMillis 触发一次检查
    - 检查druid.phyTimeoutMillis (<https://github.com/alibaba/druid/issues/870>)
      - 如果连接已经建立超过druid.phyTimeoutMillis, 断开连接, 避开MySQL的连接超时?? (如果配置心跳, 可能不需要避开)
    - 如果 连接空闲时间 >= minEvictableIdleTimeMillis
      - 如果 连接数 超过 minIdle, 则关闭连接
      - 如果 连接空闲时间 > maxEvictableIdleTimeMillis, 则关闭连接
    - 如果配置了Keepalive, 并且连接空闲时间 >= keepAliveBetweenTimeMillis, 则 进行连接心跳检查
    - 检查removeAbandoned配置 (是否有连接泄露)
      - 如果连接上没有SQL正在运行, 并且 获取连接的时间 已经超过 removeAbandonedTimeoutMillis, 则关闭连接
        - 如果设置了logAbandoned, 将连接的信息输出到日志中 (error 级别)
  - 心跳检查逻辑
    - druid.mysql.usePingMethod: 设置为true, 使用COM_PING检查心跳, 超时时间 validationQueryTimeout
    - 否则, 使用validationQuery检查心跳
  - 获取连接时
    - 如果设置了testOnBorrow, 进行心跳检查
    - 如果设置了testWhileIdle, 并且连接空闲时间超过 timeBetweenEvictionRunsMillis, 进行心跳检查
  - 归还连接时
    - 如果设置了testOnReturn, 进行心跳检查

# 配置建议

  - keepAliveBetweenTimeMillis 应小于 minEvictableIdleTimeMillis, 建议设置为 minEvictableIdleTimeMillis / 2

# 特殊

  1. druid对参数处理有问题, 从 DruidDataSource 中 load property的接口读入配置, 不会将参数传输给 MySQL Validate 插件, 需要将参数读入system.Property
  2. druid对参数处理有问题, testOnReturn等参数不能通过 DruidDataSource 中 load property的接口读入配置
