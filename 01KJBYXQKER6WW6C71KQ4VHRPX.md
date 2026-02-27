---
title: 20220818 - 百胜slave延迟, 重新整理所有线索
confluence_page_id: 1933687
created_at: 2022-08-18T11:53:23+00:00
updated_at: 2022-08-21T16:29:41+00:00
---

# 之前的关联分析

[20220612 - numa blancing]

[20220623 - Top-down Microarchitecture Analysis 尝试]

[20220808 - 百胜slave延迟, 关于内存页表的相关诊断]

[20220818 - 百胜slave延迟, 关于上下文切换的诊断]

# 线索

CPU架构类型: Skylake, 会出现 ut_delay 的单次时间比 其他CPU架构下 要长, 导致 MySQL自旋锁时间长: 

<http://10.186.18.11/confluence/pages/viewpage.action?pageId=29693988>

有uproxy向slave建立 10k+ 空闲连接, 去除空闲连接后, 延迟会快速消失: 

![image2022-8-18 19:15:18.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A15%3A18.png)

禁用部分机器的cstate (Intel Idle driver), 延迟不再发生: <https://docs.azul.com/prime/ZST-Disabling-Intel-Idle-Driver>

进程 perf:

  - 延迟增高时: 
    - ![image2022-8-18 19:18:13.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A18%3A13.png)
    - IPC下降一半
  - 将空闲连接去掉, 延迟消失
    - ![image2022-8-18 19:34:25.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A34%3A25.png)
  - 6月28日再比较
    - 无延时，11:00
      - ![image2022-8-18 19:37:59.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A37%3A59.png)
    - 11:18 15s延时
      - ![image2022-8-18 19:38:59.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A38%3A59.png)

perf top: 

  - 延迟增高时: 
    - ![image2022-8-18 19:31:55.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A31%3A55.png)
  - 将空闲连接去掉, 延迟消失
    - ![image2022-8-18 19:32:52.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A32%3A52.png)
  - 113.12 无延时的库
    - ![image2022-8-18 19:36:37.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A36%3A37.png)

node-load-misses

  - ![image2022-8-18 19:48:37.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A48%3A37.png)
  - 与同机器上的mysql比较
    - ![image2022-8-19 0:4:14.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-19%200%3A4%3A14.png)
  - 调整 innodb-numa-interleave, 拿掉空闲连接
    - ![image2022-8-18 20:11:43.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2020%3A11%3A43.png)

numa分布: 

  - ![image2022-8-18 19:50:57.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A50%3A57.png)
  - ![image2022-8-18 19:51:11.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A51%3A11.png)
  - 启用 innodb numa interleave, 但未形成均分: 
    - ![image2022-8-18 19:53:7.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A53%3A7.png)

dcycles / cycles

  - 有延迟的slave, 比例为10%, 239.26
    - ![image2022-8-18 19:55:15.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A55%3A15.png)
    - ![image2022-8-18 19:57:12.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A57%3A12.png)
    - ![image2022-8-18 19:57:30.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A57%3A30.png)
  - 239.26 无延迟时, IPC变高, dcycles 比例维持10%
    - ![image2022-8-18 20:6:32.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2020%3A6%3A32.png)
  - 239.26 , 拿掉所有空闲连接, dcycles 比例下降到6%
    - ![image2022-8-18 20:9:11.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2020%3A9%3A11.png)
    - ![image2022-8-18 21:30:5.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2021%3A30%3A5.png)
  - 无延迟的slave, 比例为5%
    - ![image2022-8-18 19:56:19.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-18%2019%3A56%3A19.png)

开着THP: 

![image2022-8-19 0:1:10.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-19%200%3A1%3A10.png)

页表性能数据: [百胜延迟perf数据.xlsx](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/%E7%99%BE%E8%83%9C%E5%BB%B6%E8%BF%9Fperf%E6%95%B0%E6%8D%AE.xlsx)

VmPTE 页表大小: 

  - 正常slave (调整过cstate)
    - ![image2022-8-20 0:11:52.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-20%200%3A11%3A52.png)
  - 异常slave:
    - ![image2022-8-20 0:12:5.png](/assets/01KJBYXQKER6WW6C71KQ4VHRPX/image2022-8-20%200%3A12%3A5.png)

# TODO

  - <https://en.pingcap.com/blog/transparent-huge-pages-why-we-disable-it-for-databases/>
    - 检查系统load, 按照检查单检查内存碎片
