---
note: 01KJBZ3M9TD227EZCG0AEGCQXE.md
title: 20231031 - mac osx 监听文件变更并同步到服务器
indexed_at: 2026-03-05T09:12:16.413150+00:00
---

## 摘要
使用 fswatch 监听文件变更，结合 xargs 和 scp 命令实现文件自动同步到远程服务器。通过管道将变更文件路径传递给 scp，实现实时上传到指定服务器路径。

## 关键概念
- fswatch: 跨平台文件变更监听工具，监控文件系统事件
- scp: 基于 SSH 的安全文件复制命令，用于远程文件传输
- xargs: 将标准输入转换为命令行参数的工具，实现命令链式调用

## 关联笔记
- 01KJC029N8APNRN6BXH70S5YQD.md: 包含 macOS 上 SSH 隧道和监听相关配置
- 01KJBZX25RKXAHW2N026JFEN5X.md: 涉及 scp 命令在服务器间传输文件的用法
