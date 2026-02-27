---
title: 20240720 - 搜乐城的wifi配置
confluence_page_id: 3145797
created_at: 2024-07-20T07:03:23+00:00
updated_at: 2024-07-30T18:11:39+00:00
---

MikroTik路由器, 需要使用winbox进行管理 (在Mac上用wine进行引导).

用IP无法连接MikroTik路由器, 需要使用mac地址, 并且使用legacy模式进行连接后管理

![image2024-7-20 14:58:36.png](/assets/01KJBZAS0XYK12NP2JGXN0WFP6/image2024-7-20%2014%3A58%3A36.png)

路由器通过ether2 连到上一级电信的路由器.

电信路由器的网段: 192.168.71.1

电信路由器的标签:

![image2024-7-20 15:2:1.png](/assets/01KJBZAS0XYK12NP2JGXN0WFP6/image2024-7-20%2015%3A2%3A1.png)

连接到无线网络, 使用手机App (网络管家) 可管理路由器 (没有通用的路由器界面)

# MikroTik配置

先将CAP配置关掉: 

![image2024-7-31 2:7:57.png](/assets/01KJBZAS0XYK12NP2JGXN0WFP6/image2024-7-31%202%3A7%3A57.png)

通过quick setup进行配置

![image2024-7-31 2:7:15.png](/assets/01KJBZAS0XYK12NP2JGXN0WFP6/image2024-7-31%202%3A7%3A15.png)

需要手工指定bridge的IP

![image2024-7-31 2:7:34.png](/assets/01KJBZAS0XYK12NP2JGXN0WFP6/image2024-7-31%202%3A7%3A34.png)

然后需要等待一段时间? 网络才能配置好. (反复配置, 通过MikroTik中的工具进行ping测试, 都无效)

完整的配置文件: 

[附件: 20240731.cnf.rsc] 

# Nas

![image2024-7-31 2:10:29.png](/assets/01KJBZAS0XYK12NP2JGXN0WFP6/image2024-7-31%202%3A10%3A29.png)
