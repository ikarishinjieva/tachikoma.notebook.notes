---
note: 01KJBYFB4SZZD99K9EB5466GC2.md
title: 20211116 - 将进程移到screen中执行
indexed_at: 2026-03-05T07:50:09.470555+00:00
---

## 摘要
记录将已运行进程迁移到 screen 会话的完整操作流程，包括使用 ctrl+z、bg、disown 解除进程与 TTY 关联，再通过 screen 和 reptyr 重新挂载。遇到 reptyr 因进程存在子进程而附加失败的报错，尝试使用-T 参数解决。

## 关键概念
- bg: 将暂停的进程移到后台继续运行
- disown: 解除进程与当前 shell 的关联，防止退出时进程被终止
- reptyr: 将已运行的进程重新关联到新的 TTY 或 screen 会话
- screen: 终端复用工具，可创建持久的终端会话

## 关联笔记
- 01KJBZJT9YFKP1XPAKC0PHG4GA.md: 在 screen 会话中运行深度学习训练任务的实际应用示例
- 01KJBZD8CNV250ZM6CJSSVHKYH.md: 使用 py-spy 查看 Python 进程堆栈，与进程管理相关
