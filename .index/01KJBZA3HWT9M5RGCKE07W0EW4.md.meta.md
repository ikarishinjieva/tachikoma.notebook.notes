---
note: 01KJBZA3HWT9M5RGCKE07W0EW4.md
title: 20240614 - 阅读论文: RepoCoder: Repository-Level Code Completion Through Iterative Retrieval and Generation
indexed_at: 2026-03-05T10:10:18.587604+00:00
---

## 摘要
论文提出 RepoCoder，通过迭代式代码切片结合 RAG 检索实现仓库级代码补全。每个迭代处理一个代码切片，允许与上一切片重叠，重叠部分使用前次迭代的优化结果作为已知上下文。该方法创新有限，核心是将代码分片并逐片迭代优化。

## 关键概念
- 迭代式代码补全: 将代码补全任务分解为多个迭代步骤，逐步完成
- 代码切片: 将完整代码分割成多个片段，每片允许与上一片部分重叠
- RAG 检索: 对每个切片检索仓库中相似代码片段作为补全参考
- 重叠优化: 下一迭代的重叠部分携带上一迭代的优化结果作为已知信息

## 关联笔记
- 01KJBZR6R18RZDCX0A1GJ7HMB5.md: 同样研究代码生成任务，使用 MCTS 进行迭代式推理优化
- 01KJBZG3NMEV9RPF9NN5VBTQ9Z.md: RAGCHECKER 框架，提供 RAG 系统细粒度评估方法，可评估此类代码 RAG 系统
