---
note: 01KJBYF8NABH5KTYVP828JKC9V.md
title: 20211109 - 如何运行 ida-free (xterm)
indexed_at: 2026-03-05T07:49:10.988983+00:00
---

## 标签
IDA free, X11 forwarding, SSH, xterm, GUI 应用，远程调试

## 摘要
解决 IDA free 在远程服务器运行时因缺少 X11 显示导致的 QXcbConnection 错误。通过在服务器安装 xterm，客户端使用 SSH X11 转发 (ssh -XY) 连接并启动图形界面应用。

## 关键概念
- X11 Forwarding: SSH 协议支持将远程 X11 图形界面转发到本地显示
- ssh -XY: 启用 X11 转发的 SSH 连接命令，-X 启用转发，-Y 启用可信转发
- xterm: X Window 系统的终端模拟器，用于在远程服务器提供 GUI 环境

## 关联笔记
- 01KJBYFBDSR1SNE0X711Z0TGXR.md: 同日期笔记，涉及 IDAPro 和 anvill 反编译工具链使用
- 01KJBZEMNJW3ZAV821WQ5N70HB.md: 包含类似的 ssh -XY xterm 命令，用于远程 GUI 应用启动
- 01KJBYF973H98W1RNZ05SS50KE.md: 涉及 IDAPro/reopt 等反编译工具，属于同一逆向工程工作流
