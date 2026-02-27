---
note: 01KJBZ9MSBPXE97PBVX7FSVN5P.md
title: 20240605 - Mac下打zip, 需要排除__MAXOSX文件夹
indexed_at: 2026-02-27T10:56:29.246201+00:00
---

用户要求我分析一个关于 Mac 下 zip 命令排除__MACOSX 文件夹的笔记，并生成索引元数据。

让我分析这个笔记：
1. 标题：20240605 - Mac 下打 zip, 需要排除__MAXOSX 文件夹
2. 内容：一个 zip 命令，用于压缩时排除__MACOSX 文件夹

标签应该包括：macOS, zip, 压缩，命令行，文件管理

摘要：记录在 macOS 系统下使用 zip 命令压缩文件时，排除__MACOSX 隐藏文件夹的方法。

关键概念：
- zip 命令：Unix/Linux 下的压缩工具
- __MACOSX：macOS 创建的隐藏文件夹，存储资源叉信息
- -x 参数：zip 命令的排除选项

关联笔记：查看提供的笔记列表，主要是关于 MySQL、Clickhouse、数据库、性能分析等内容，与这个 zip 命令笔记没有明显关联。

让我按照格式输出。
## 标签
macOS, zip, 压缩，命令行，文件排除

## 摘要
记录在 macOS 系统下使用 zip 命令压缩文件时排除__MACOSX 隐藏文件夹的方法。__MACOSX 是 macOS 自动生成的元数据文件夹，通常不需要打包分发。

## 关键概念
- zip -r: 递归压缩目录及其子目录
- -x 参数: 排除指定模式的文件或目录
- __MACOSX: macOS 创建的隐藏文件夹，存储资源叉和元数据

## 关联笔记
无
