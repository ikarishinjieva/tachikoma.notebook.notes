---
title: 20211108 - gdb追踪fork出的进程
confluence_page_id: 1572970
created_at: 2021-11-08T05:24:08+00:00
updated_at: 2021-11-08T05:43:44+00:00
---

break main

set detach-on-fork off

set follow-fork-mode child

可追踪fork出的进程的main方法

注意: gdb 8 会报错, 升级到 gdb 10.2 后解决
