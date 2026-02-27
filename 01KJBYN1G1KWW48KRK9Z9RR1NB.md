---
title: 20220915 - 电信IO分析 blktrace
confluence_page_id: 1933812
created_at: 2022-09-15T05:52:53+00:00
updated_at: 2022-09-15T08:59:03+00:00
---

参考: <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/performance_tuning_guide/ch06s03>

正常: 

![image2022-9-15 13:50:42.png](/assets/01KJBYN1G1KWW48KRK9Z9RR1NB/image2022-9-15%2013%3A50%3A42.png)

异常: 

![image2022-9-15 13:51:0.png](/assets/01KJBYN1G1KWW48KRK9Z9RR1NB/image2022-9-15%2013%3A51%3A0.png)

D2C显示往IO设备交互的延迟高

获取btt 的人类可读输出: 

[blk2930](/assets/01KJBYN1G1KWW48KRK9Z9RR1NB/blk2930)

![image2022-9-15 16:55:21.png](/assets/01KJBYN1G1KWW48KRK9Z9RR1NB/image2022-9-15%2016%3A55%3A21.png)

"这一片IO, 有不同时间发出的D, 都在同一时间完成C, 也就是说设备看上去是卡在某一个时间就突然活过来了"
