---
title: 20210923 - hash join 使用文件句柄的数量
confluence_page_id: 1343742
created_at: 2021-09-23T13:02:19+00:00
updated_at: 2021-09-23T13:02:19+00:00
---

# 技术背景

<http://mysql.taobao.org/monthly/2019/11/02/>

  - MySQL 实现 的 Hash Join 为Hybrid Hash Join, 将 join 的两张表 切分为chunk, 放在磁盘中, 分别为: build table 和 probe table
  - 每一种chunk table, 均有128个文件, 共256个文件
  - 一个build table 和 一个 probe table 被加载进内存, 在内存中进行hash join, 产生结果
  - 如何进行chunk切分??

# 一问一实验

发现hash join产生大量文件句柄

使用perf最终文件句柄跟谁有关: 

perf record -e syscalls:sys_exit_openat -p 12340 --call-graph dwarf,4096

perf script
