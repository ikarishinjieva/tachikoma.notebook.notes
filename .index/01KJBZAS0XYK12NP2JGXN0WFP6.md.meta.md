---
note: 01KJBZAS0XYK12NP2JGXN0WFP6.md
title: 20240720 - 搜乐城的wifi配置
indexed_at: 2026-03-05T10:23:10.868340+00:00
---

## 标签
MikroTik, 路由器配置, winbox, 网络管理, NAS, 电信路由器

## 摘要
记录搜乐城的 MikroTik 路由器配置过程，包括使用 winbox 通过 MAC 地址和 legacy 模式连接管理、bridge IP 手工指定、CAP 配置关闭等步骤。路由器通过 ether2 连接电信上级路由器 (192.168.71.1)，无线网络可通过网络管家 App 管理。

## 关键概念
- MikroTik 路由器: 需要使用 winbox 工具进行管理的网络设备
- winbox: MikroTik 官方管理工具，在 Mac 上需通过 wine 运行
- Bridge 配置: 需要手工指定 IP 地址的网络桥接设置
- CAP 配置: 路由器中的无线接入点控制协议，配置时需关闭

## 关联笔记
无
