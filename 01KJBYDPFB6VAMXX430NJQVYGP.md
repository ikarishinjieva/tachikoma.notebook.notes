---
title: 20210710 - 华北第二个vpn无法ping通公司网段
confluence_page_id: 1147070
created_at: 2021-07-10T15:26:58+00:00
updated_at: 2021-07-10T16:27:16+00:00
---

在阿里云上服务器, 假设vpn client现象相同, openvpn通畅但无法ping通10.186.18.*网段

对比:

  1. 从第一个vpn进行ping, 在openvpn server端抓包, 可见 正常情况:   
  

```
23:14:07.257339  In ethertype IPv4 (0x0800), length 100: (tos 0x0, ttl 63, id 47063, offset 0, flags [DF], proto ICMP (1), length 84)
    192.168.88.180 > 10.186.18.20: ICMP echo request, id 78, seq 51, length 64
23:14:07.257377 Out 12:8f:f6:c4:3f:60 ethertype IPv4 (0x0800), length 100: (tos 0x0, ttl 62, id 47063, offset 0, flags [DF], proto ICMP (1), length 84)
    192.168.88.180 > 10.186.18.20: ICMP echo request, id 78, seq 51, length 64
23:14:07.257608  In ac:8d:34:eb:00:ae ethertype IPv4 (0x0800), length 100: (tos 0x0, ttl 63, id 26396, offset 0, flags [none], proto ICMP (1), length 84)
    10.186.18.20 > 192.168.88.180: ICMP echo reply, id 78, seq 51, length 64
23:14:07.257619 Out ethertype IPv4 (0x0800), length 100: (tos 0x0, ttl 62, id 26396, offset 0, flags [none], proto ICMP (1), length 84)
    10.186.18.20 > 192.168.88.180: ICMP echo reply, id 78, seq 51, length 64
``` 
  2. 从第二个vpn进行ping, 在openvpn server端抓包, 可见异常情况:   
  

```
23:26:13.981389  In ethertype IPv4 (0x0800), length 100: (tos 0x0, ttl 64, id 31037, offset 0, flags [DF], proto ICMP (1), length 84)
    172.16.12.2 > 10.186.18.20: ICMP echo request, id 14183, seq 1, length 64
23:26:13.981409 Out 12:8f:f6:c4:3f:60 ethertype IPv4 (0x0800), length 100: (tos 0x0, ttl 63, id 31037, offset 0, flags [DF], proto ICMP (1), length 84)
    172.16.12.2 > 10.186.18.20: ICMP echo request, id 14183, seq 1, length 64
``` 

发现源包地址不同, 正常情况是 ping发起端的地址 (是从非VPN client的机器上发起), 异常情况 是 VPN peer的地址 (是从VPN client的机器上发起)

在VPN client端, 可以通过 "ping -I 172.22.210.100 10.186.18.20" 来指定来源IP, 使得来源IP不是VPN peer的地址. 

并将VPN server的配置中的route设置为: "route 172.22.210.0 255.255.255.0"

使得异常情况和正常情况的源包地址类似, 但仍然无法ping通, 抓包如下: 

```
23:40:51.116152  In ethertype IPv4 (0x0800), length 100: (tos 0x0, ttl 64, id 39435, offset 0, flags [DF], proto ICMP (1), length 84)
    172.22.210.100 > 10.186.18.20: ICMP echo request, id 14203, seq 109, length 64
23:40:51.116189 Out 12:8f:f6:c4:3f:60 ethertype IPv4 (0x0800), length 100: (tos 0x0, ttl 63, id 39435, offset 0, flags [DF], proto ICMP (1), length 84)
    172.22.210.100 > 10.186.18.20: ICMP echo request, id 14203, seq 109, length 64
``` 

在10.186.18.20 (PING的目标机上抓包): 

正常情况: 

```
23:42:42.952347 12:8f:f6:c4:3f:60 > a2:8e:50:80:ca:bf, ethertype IPv4 (0x0800), length 98: (tos 0x0, ttl 62, id 60872, offset 0, flags [DF], proto ICMP (1), length 84)
    192.168.88.180 > 10.186.18.20: ICMP echo request, id 79, seq 1, length 64
23:42:42.952428 a2:8e:50:80:ca:bf > ac:8d:34:eb:00:ae, ethertype IPv4 (0x0800), length 98: (tos 0x0, ttl 64, id 69, offset 0, flags [none], proto ICMP (1), length 84)
    10.186.18.20 > 192.168.88.180: ICMP echo reply, id 79, seq 1, length 64
``` 

  
异常情况: 

```
23:43:00.604822 12:8f:f6:c4:3f:60 > a2:8e:50:80:ca:bf, ethertype IPv4 (0x0800), length 98: (tos 0x0, ttl 63, id 52120, offset 0, flags [DF], proto ICMP (1), length 84)
    172.22.210.100 > 10.186.18.20: ICMP echo request, id 14207, seq 6, length 64
23:43:00.604875 a2:8e:50:80:ca:bf > ac:8d:34:eb:00:ae, ethertype IPv4 (0x0800), length 98: (tos 0x0, ttl 64, id 4800, offset 0, flags [none], proto ICMP (1), length 84)
    10.186.18.20 > 172.22.210.100: ICMP echo reply, id 14207, seq 6, length 64
``` 

可见目标端都已经接到PING, 并且正确回复, 但VPN server端接不到回复

猜测 192.168.88.0 网段, 在10.186.18.1的路由规则里, 已经配置了转发到10.186.18.22. 而172.22.210.100未配置规则被丢弃.

更换192.168.88.180 作为 VPN client, 重做实验: 

将VPN server配置文件改为: "route 192.168.88.180 255.255.255.255", 实验可以成功

现在要解决的问题是: 实验中用ping更换了源端IP, 在默认应用中也需要有更换源端IP的操作? 

在VPN client配置防火墙规则: 

```
iptables -t nat -A POSTROUTING --destination 10.186.0.0/16 -j SNAT --to-source 192.168.88.180
``` 

更换包的源地址, 可实验通过
