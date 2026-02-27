---
note: 01KJBZ3M9TD227EZCG0AEGCQXE.md
title: 20231031 - mac osx 监听文件变更并同步到服务器
indexed_at: 2026-02-27T10:37:30.900501+00:00
---

用户要求我分析一个笔记并生成索引元数据。让我先理解笔记内容：

标题：20231031 - mac osx 监听文件变更并同步到服务器

内容：一个使用 fswatch 监听文件变更并通过 scp 同步到远程服务器的命令。

这是一个关于文件同步/自动化的技术笔记。

让我按要求生成：

## 标签
macOS, 文件监控, 自动同步, fswatch, scp, 远程部署

## 摘要
记录了在 macOS 上使用 fswatch 监听文件变更并自动同步到远程服务器的方法。通过管道组合 fswatch 和 xargs、scp 实现一键实时同步。

## 关键概念
- fswatch: macOS 文件变化监控工具
- xargs: 将标准输入转换为命令行参数
- scp: 基于 SSH 的文件传输命令

## 关联笔记
看其他笔记列表，主要是关于 MySQL、Clickhouse、性能分析等，没有直接相关的文件同步或 macOS 相关的笔记。所以应该是「无」。
## 标签
macOS, 文件监控, 自动同步, fswatch, scp, 远程部署

## 摘要
记录了在 macOS 上使用 fswatch 监听文件变更并自动同步到远程服务器的方法。通过管道组合 fswatch、xargs 和 scp 实现一键实时同步。

## 关键概念
- fswatch: macOS 文件变化监控工具，可实时检测文件修改
- xargs: 将标准输入转换为命令行参数执行
- scp: 基于 SSH 的安全文件传输命令

## 关联笔记
无
