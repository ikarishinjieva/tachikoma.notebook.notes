---
title: 20210716 - CPU max MHz 的解释
confluence_page_id: 1343498
created_at: 2021-07-16T08:52:12+00:00
updated_at: 2021-07-16T09:20:22+00:00
---

<https://github.com/karelzak/util-linux/blob/master/sys-utils/lscpu.c>

max CPU MHz的来源为 cpufreq

以117为例: 

  - cpuinfo
    - ![image2021-7-16 16:44:56.png](/assets/01KJBYDPJ1GJ9A3G56J24E70BM/image2021-7-16%2016%3A44%3A56.png)
  - cpufreq
    - ![image2021-7-16 16:48:56.png](/assets/01KJBYDPJ1GJ9A3G56J24E70BM/image2021-7-16%2016%3A48%3A56.png)
  - CPU当前频率为1.2 GHz
  - 最高频率为 3.6 GHz, 但CPU型号为 2.2 GHz

查询Intel官网: 

<https://ark.intel.com/content/www/us/en/ark/products/91753/intel-xeon-processor-e5-2698-v4-50m-cache-2-20-ghz.html>

![image2021-7-16 16:53:36.png](/assets/01KJBYDPJ1GJ9A3G56J24E70BM/image2021-7-16%2016%3A53%3A36.png)

参考文档: <https://wiki.archlinux.org/title/CPU_frequency_scaling#Setting_maximum_and_minimum_frequencies>

![image2021-7-16 17:20:20.png](/assets/01KJBYDPJ1GJ9A3G56J24E70BM/image2021-7-16%2017%3A20%3A20.png)
