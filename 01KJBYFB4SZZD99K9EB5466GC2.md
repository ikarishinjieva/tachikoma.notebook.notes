---
title: 20211116 - 将进程移到screen中执行
confluence_page_id: 1573050
created_at: 2021-11-16T10:26:24+00:00
updated_at: 2021-11-16T10:28:28+00:00
---

1. ctrl+z 停止进程
  2. bg 将进程移到后台
  3. disown %1 解除进程与tty的关联
  4. 退出tty相关bash

  5. 运行screen
  6. reptyr {pid} 重新关联进程

  

获得报错: 

```
root@ubuntu:/opt# reptyr 2987
[-] Process 2988 (sh) shares 2987's process group. Unable to attach.
(This most commonly means that 2987 has suprocesses).
Unable to attach to pid 2987: Invalid argument
``` 

  

尝试reptyr -T ??
