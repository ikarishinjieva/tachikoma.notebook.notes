---
title: 20220826 - 百胜slave延迟, sar探索
confluence_page_id: 1933731
created_at: 2022-08-25T16:15:16+00:00
updated_at: 2022-09-07T06:27:41+00:00
---

获取sar: 

[附件: 239-26-sar.tar.gz] 

# 延迟

20日 延迟图: 

![image2022-8-26 0:9:47.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-26%200%3A9%3A47.png)

![image2022-8-26 0:10:0.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-26%200%3A10%3A0.png)

21日延迟图: 

![image2022-8-26 0:10:33.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-26%200%3A10%3A33.png)

26日/27日 延迟图: 

![image2022-8-27 22:59:35.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-27%2022%3A59%3A35.png)

# 上下文切换

21日上下文切换图: 

![image2022-8-26 0:12:36.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-26%200%3A12%3A36.png)

![image2022-8-26 0:14:23.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-26%200%3A14%3A23.png)

20日上下文切换图: 

![image2022-8-26 0:13:36.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-26%200%3A13%3A36.png)

![image2022-8-26 0:14:0.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-26%200%3A14%3A0.png)

# pgpgout/s

20日

![image2022-8-26 15:1:57.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-26%2015%3A1%3A57.png)

  - 没有pgpgin

26日

![image2022-8-27 22:59:8.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-27%2022%3A59%3A8.png)

  - pgpgin与pgpgout趋势一致

27日

![image2022-8-27 22:59:21.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-27%2022%3A59%3A21.png)

  - pgpgin与pgpgout趋势一致

参考: 

  - <https://access.redhat.com/solutions/21526>
    - 提到用systemtap脚本 (猜测是 memory/pfaults.stp ), 来观测缺页异常

扫描Linux源码, 找到 PGPGOUT的位置: 

![image2022-8-27 23:20:43.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-8-27%2023%3A20%3A43.png)

可尝试使用 bcc 来观测 submit_bio 的调用: <https://github.com/iovisor/bcc/blob/e83019bdf6c400b589e69c7d18092e38088f89a8/tools/stackcount_example.txt>

# CPU异常

20日

在11:17分左右, 发生 偶数CPU -> 奇数CPU 的切换

![image2022-9-6 23:23:9.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A23%3A9.png)

在 22:55 - 22:56分左右, 发生 奇数CPU -> 偶数CPU 的切换

![image2022-9-6 23:22:0.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A22%3A0.png)

两个时间与20日的 延迟起止时间重合

![image2022-9-6 23:36:23.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A36%3A23.png)

其他20日的sar图: 

![image2022-9-6 23:27:16.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A27%3A16.png)

pgpgout 的峰值是延迟曲线上升的起点.

fault 在整个延迟过程中变低

![image2022-9-6 23:29:53.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A29%3A53.png)

bwrtn/s 与 pgpgout趋势相同

![image2022-9-6 23:37:39.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A37%3A39.png)

wtps和tps与pgpgout趋势相同

![image2022-9-6 23:38:19.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A38%3A19.png)

![image2022-9-6 23:38:57.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A38%3A57.png)

kbmemused

![image2022-9-6 23:46:45.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A46%3A45.png)

dentunusd 出现下滑和回升, 程度不明显

![image2022-9-6 23:48:58.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A48%3A58.png)

ldavg-15

![image2022-9-6 23:51:1.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A51%3A1.png)

wr_sec/s

![image2022-9-6 23:54:13.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A54%3A13.png)

avgrq-sz: IO请求的数据量大小 在 写流量增大时 也增大

![image2022-9-6 23:55:3.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A55%3A3.png)

avgqu-sz/await/svctm

写入量越多的时候, 三项值都在变低? IO效率在变高?

avgqu-sz: IO请求队列长度 变低

await/svctm: IO延迟指标 变低

![image2022-9-7 13:2:12.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-7%2013%3A2%3A12.png)

await: 

参考: <https://access.redhat.com/articles/524353>

![image2022-9-6 23:56:1.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A56%3A1.png)

svctm

![image2022-9-6 23:59:1.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-6%2023%3A59%3A1.png)

%util

![image2022-9-7 0:0:5.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-7%200%3A0%3A5.png)

网络: 

![image2022-9-7 0:2:20.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-7%200%3A2%3A20.png)

图表源文件: 

[附件: sa20.xlsx] 

# 取同组slave 29日监控

![image2022-9-7 14:17:26.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-7%2014%3A17%3A26.png)

[113-15-sa29.txt](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/113-15-sa29.txt)

CPU未发生 奇偶数切换

![image2022-9-7 14:22:6.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-7%2014%3A22%3A6.png)

![image2022-9-7 14:21:47.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-7%2014%3A21%3A47.png)

![image2022-9-7 14:22:26.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-7%2014%3A22%3A26.png)

![image2022-9-7 14:27:17.png](/assets/01KJBYXQEJ0X39WB95B5E7PMN3/image2022-9-7%2014%3A27%3A17.png)

未见明显的fault下降
