---
title: 20210404 - 如何给LWP降速
confluence_page_id: 753785
created_at: 2021-04-04T02:42:18+00:00
updated_at: 2021-04-12T04:01:43+00:00
---

# 尝试1: cpulimit 原理研究

cpulimit 从 procfs 中读取进程的cpu_time

达到限速条件, 对进程进行SIGSTOP, 低于限速条件, 对进城进行SIGCONT, 以对进程进行限速

经过测试, 即使SIGSTOP发送给线程, 也会让进程完全停止

# 尝试2

来源: <https://comp.programming.threads.narkive.com/JUduyL4B/pthread-kill-tid-sigstop-for-async-thread-suspending>

If you are looking for such a capability - also something similar to the  
Solaris UI thr_suspend() / thr_continue() ) - then have a look at Dave's  
anthology "Programming with POSIX threads" §6.6.3 pp 217-227

在Programming with POSIX threads中对应章节, 找到如下描述: 

Remember that, aside from signal-catching functions, signal actions affect the process.

Sending the SIGKILL signal to a specific thread using pthread_kill will kill the process, not just

the specified thread. Use pthread_cancel to get rid of a particular thread (see Section 5.3). Sending

SIGSTOP to a thread will stop all threads in the process until a SIGCONT is sent by some other

process.

方案失败

# GDB 停止单个线程

[20210404 - gdb只停止触发断点的线程]
