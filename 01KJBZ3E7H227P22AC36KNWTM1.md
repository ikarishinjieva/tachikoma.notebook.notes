---
title: 20231024 - arthas 学习
confluence_page_id: 2589264
created_at: 2023-10-24T11:55:07+00:00
updated_at: 2023-10-27T06:14:27+00:00
---

有用的功能: 

  - thread : <https://arthas.aliyun.com/en/doc/thread.html>
    - 阻塞其他线程的线程: locate the thread blocking the others. 机制?
    - 最繁忙的n个线程: the top n busiest threads with stack traces printed.
  - vmtool : <https://arthas.aliyun.com/en/doc/vmtool.html>
    - 支持 getInstances
    - 支持 forceGc
  - dump: <https://arthas.aliyun.com/en/doc/dump.html>
    - dump 类的二进制代码
  - mc: <https://arthas.aliyun.com/en/doc/mc.html>
    - 带环境的编译? 
  - retransform: <https://arthas.aliyun.com/en/doc/retransform.html>
    - 替换class
  - sc: <https://arthas.aliyun.com/en/doc/sc.html>
    - 搜索class
  - trace: <https://arthas.aliyun.com/en/doc/trace.html>
    - 给入口方法, 追踪入口方法内的调用内容, 统计方法metric
  - tt: <https://arthas.aliyun.com/en/doc/tt.html>
    - 记录函数调用的上下文, 可追踪当时记录的变量值等
  - watch: <https://arthas.aliyun.com/en/doc/watch.html>
    - Aspect切片
