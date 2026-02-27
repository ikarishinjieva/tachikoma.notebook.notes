---
title: 20211109 - 如何运行 ida-free (xterm)
confluence_page_id: 1573015
created_at: 2021-11-09T10:30:46+00:00
updated_at: 2021-11-19T05:25:29+00:00
---

IDA free带有界面, 在shell中运行会报错: QXcbConnection: Could not connect to display

解决: 

根据 <https://stephon.pixnet.net/blog/post/45160896-%5Bubuntu%5D-x11-forwarding-with-xvfb-to-run-remote-x-applicatio>

在server端: 

  1. 安装xterm  
  

在client端:

  1. 安装xquatz
  2. 执行: ssh -XY [a.blah.example.com](<http://a.blah.example.com>) -l stephon xterm
