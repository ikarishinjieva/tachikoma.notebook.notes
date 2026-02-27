---
title: 20210406 - innodb 如何后台不刷脏页, 前台会怎么处理
confluence_page_id: 753801
created_at: 2021-04-06T03:36:31+00:00
updated_at: 2021-04-13T07:20:06+00:00
---

# 原始信息

gdb /root/opt/mysql/5.7.20/bin/mysqld

set args --defaults-file=/root/sandboxes/msb_5_7_20/my.sandbox.cnf --basedir=/root/opt/mysql/5.7.20 --datadir=/root/sandboxes/msb_5_7_20/data --plugin-dir=/root/opt/mysql/5.7.20/lib/plugin --user=root --log-error=/root/sandboxes/msb_5_7_20/data/msandbox.err --pid-file=/root/sandboxes/msb_5_7_20/data/mysql_sandbox5720.pid --socket=/tmp/mysql_sandbox5720.sock --port=5720

b pc_sleep_if_needed thread 11

interrupt -a

thread apply all bt

log_checkpoint_margin 用于检查是否需要同步的 preflush 和 checkpoint

srv_flush_sync 在SQL处理中, 是否强制唤醒page cleaner进行IO burst

打开innodb monitor: 

SELECT name, subsystem, status FROM INFORMATION_SCHEMA.INNODB_METRICS ORDER BY NAME;

SET GLOBAL innodb_monitor_enable = 'log%';

log_lsn_checkpoint_age 与 log_max_modified_age_async/log_max_modified_age_sync 进行比较

# 结论

已更新一问一实验
