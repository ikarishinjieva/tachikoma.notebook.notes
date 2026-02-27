---
title: 20231121 - async-profiler在容器中抓取java的火焰图
confluence_page_id: 2589616
created_at: 2023-11-21T07:29:54+00:00
updated_at: 2023-11-21T07:29:54+00:00
---

```
./profiler.sh start 44853 --fdtransfer
 
./profiler.sh status 44853 --fdtransfer
 
./profiler.sh stop 44853 --fdtransfer -o flamegraph
```
