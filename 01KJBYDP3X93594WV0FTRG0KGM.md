---
title: 20210705 - linux的CPU频率
confluence_page_id: 1147061
created_at: 2021-07-05T07:50:10+00:00
updated_at: 2021-07-05T08:40:28+00:00
---

/proc/stat中CPU使用率的单位为 USER_HZ, 查看: 

```
root@ubuntu:~# cat /proc/sys/net/ipv4/neigh/lo/locktime
100
``` 

<https://www.kernel.org/doc/html/latest/filesystems/proc.html#proc-pid-timerslack-ns-task-timerslack-value>

![image2021-7-5 16:36:59.png](/assets/01KJBYDP3X93594WV0FTRG0KGM/image2021-7-5%2016%3A36%3A59.png)

\---

/proc/pid/stat中CPU使用率的单位为 clock tick (_SC_CLK_TCK), 查看: 

```
root@ubuntu:~# grep 'CONFIG_HZ=' /boot/config-$(uname -r)
CONFIG_HZ=250
``` 

![image2021-7-5 16:40:22.png](/assets/01KJBYDP3X93594WV0FTRG0KGM/image2021-7-5%2016%3A40%3A22.png)
