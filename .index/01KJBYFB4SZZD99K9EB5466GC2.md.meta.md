---
note: 01KJBYFB4SZZD99K9EB5466GC2.md
title: 20211116 - 将进程移到screen中执行
indexed_at: 2026-02-27T10:13:38.629964+00:00
---

用户要求我分析这篇关于将进程移到 screen 中执行的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记主题：关于如何将正在运行的进程移到 screen 中执行
2. 主要步骤：ctrl+z、bg、disown、screen、reptyr
3. 遇到的问题：reptyr 报错，提示进程有子进程
4. 尝试的解决方案：reptyr -T

标签应该包括：screen、进程管理、reptyr、Linux、后台进程等

摘要：描述将进程迁移到 screen 的方法和遇到的问题

关键概念：screen、reptyr、disown、进程组等

关联笔记：查看其他笔记列表，看是否有相关的。其他笔记主要是关于 MySQL、Clickhouse、gdb、内存等主题，与这篇关于 screen 和进程管理的笔记关联度不高。可能没有直接关联的笔记。

让我生成输出：## 标签
screen, 进程管理, reptyr, Linux, 后台进程, tty

## 摘要
记录将已运行进程迁移到 screen 会话的操作步骤，包括停止、后台化、解除 tty 关联后使用 reptyr 重新关联。遇到 reptyr 因进程存在子进程而附加失败的报错，尝试使用 -T 参数解决。

## 关键概念
- screen: Linux 终端复用工具，可创建独立会话运行进程
- reptyr: 将运行中的进程重新关联到新终端的工具
- disown: 解除进程与 shell 的关联，防止 shell 退出时终止进程
- 进程组: 报错显示目标进程与子进程共享进程组导致无法附加

## 关联笔记
无
