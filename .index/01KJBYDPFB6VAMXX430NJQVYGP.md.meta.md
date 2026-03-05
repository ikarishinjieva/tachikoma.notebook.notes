---
note: 01KJBYDPFB6VAMXX430NJQVYGP.md
title: 20210710 - 华北第二个vpn无法ping通公司网段
indexed_at: 2026-03-05T07:35:06.728860+00:00
---

## 标签
OpenVPN, 网络故障排查，iptables, SNAT, 路由配置，VPN

## 摘要
记录华北第二个 VPN 无法 ping 通公司 10.186.18.*网段的排查过程。通过对比正常/异常情况的抓包数据，发现源包地址差异导致路由规则不匹配，最终通过 iptables SNAT 规则更换包的源地址解决问题。

## 关键概念
- OpenVPN: 开源 VPN 解决方案，用于建立加密的网络隧道
- SNAT: 源地址转换，修改网络包的源 IP 地址


- 路由规则: 决定网络包转发路径的配置
- tcpdump: 网络抓包工具，用于分析网络流量

## 关联笔记
- 01KJC029N8APNRN6BXH70S5YQD.md: 同样涉及通过 VPN 连接公司服务器时的网络连通性问题
- 01KJC02AC69S96NKJ7VYQM1AV6.md: 包含类似的 iptables SNAT 配置用于解决 VPN 连接问题## 标签
OpenVPN, 网络故障排查，iptables, SNAT, 路由配置，VPN

## 摘要
记录华北第二个 VPN 无法 ping 通公司 10.186.18.*网段的排查过程。通过对比正常/异常情况的抓包数据，发现源包地址差异导致路由规则不匹配，最终通过 iptables SNAT 规则更换包的源地址解决问题。

## 关键概念
- OpenVPN: 开源 VPN 解决方案，用于建立加密的网络隧道
- SNAT: 源地址转换，修改网络包的源 IP 地址以匹配路由规则
- 路由规则: 决定网络包转发路径的配置
- tcpdump: 网络抓包工具，用于分析网络流量
- ICMP: 网络控制消息协议，ping 命令使用的协议

## 关联笔记
- 01KJC029N8APNRN6BXH70S5YQD.md: 同样涉及通过 VPN 连接公司服务器时的网络连通性问题
- 01KJC02AC69S96NKJ7VYQM1AV6.md: 包含类似的 iptables SNAT 配置用于解决 VPN 连接问题
