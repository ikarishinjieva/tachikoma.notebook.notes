---
note: 01KJBZ04YB9HSQ98XKQ8KF2SMF.md
title: 20230523 - 修复llama_index的refine接口
indexed_at: 2026-03-05T08:45:00.296414+00:00
---

## 标签
llama_index, refine 接口，chunk 处理，response 等待，prompt 修复

## 摘要
refine 接口在连续 chunk 处理时，后续 chunk 的 response 非字符串导致 prompt 错误。通过增加等待 response 的代码解决该问题。

## 关键概念
- refine 接口：llama_index 中用于迭代优化响应的处理机制
- chunk：文本分割后的处理单元
- response 等待：确保 response 为字符串后再继续处理的同步机制

## 关联笔记
- 01KJBZ044K0F6J2PM9E6FVJ07T.md: 同一天记录 llama_index refine 机制的另一篇笔记，探讨 refine 未被使用的原因
