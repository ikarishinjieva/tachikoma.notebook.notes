---
note: 01KJBYEXSH3QYZQX5PG6DMJ36S.md
title: 20210923 - linux rc script 相关知识
indexed_at: 2026-03-05T07:44:53.961738+00:00
---

## 标签
Linux, systemd, 启动流程, runlevel, rc script, init

## 摘要
记录 Linux 5.x 传统 SysV init 启动流程（BIOS→Boot Loader→Kernel→/sbin/init→rc.d 脚本）与 systemd 启动流程的对比。包含 systemctl list-dependencies 输出示例、systemd 与 runlevel 的对应关系，以及修改默认 target 的方法。

## 关键概念
- runlevel: Linux 传统运行级别，定义系统启动后的服务状态
- rc.d 脚本: /etc/rc.d/rcX.d/中的K(停止)/S(启动)脚本，按优先级顺序执行
- systemd target: systemd 中替代 runlevel 的目标单元，如 default.target、rescue.target
- systemctl list-dependencies: 列出指定 target 的依赖服务树
- default.target: systemd 默认启动目标，可通过 set-default 修改

## 关联笔记
- 01KJBZ9V7H1SZ8CZRJRTX1W7GM.md: 涉及修改 systemd service 配置（docker.service）和 systemctl 命令使用
- 01KJC08NBRWP9P7TG4MVZS84H8.md: 包含 systemd 服务启动日志示例（dbus-daemon、dcvsessionlauncher.service）
