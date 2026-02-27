---
note: 01KJBZKH1PPBTSN0YMS3D69E2N.md
title: 20241115 - ms-swift是如何计算acc的
indexed_at: 2026-02-27T11:22:08.872170+00:00
---

用户要求我分析一篇关于 ms-swift 如何计算准确率的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记主题：关于 `compute_acc_metrics` 函数的分析，这是用于计算生成式模型准确率的函数
2. 核心内容：
   - 函数支持 token 级别和 sentence 级别的准确率计算
   - 处理编码器 - 解码器模型和非编码器 - 解码器模型
   - 处理 padding tokens（-100 掩码）
   - 结论：sentence 模式评估句子和预期完全一样，token 模式评估 token 和预期完全一样

3. 标签应该是：机器学习、准确率计算、模型评估、生成式模型、ms-swift 等

4. 关键概念：
   - acc_strategy: 准确率计算策略（token 或 sentence）
   - 编码器 - 解码器模型
   - 掩码（mask）
   - token 级别准确率
   - 句子级别准确率

5. 关联笔记：查看提供的笔记列表，这些都是关于数据库、系统性能、Clickhouse 等的笔记，与这篇机器学习/模型评估的笔记没有明显关联。所以应该写「无」。

让我按照要求的格式输出。## 标签
机器学习，模型评估，准确率计算，生成式模型，ms-swift

## 摘要
分析 ms-swift 框架中 `compute_acc_metrics` 函数的实现逻辑，支持 token 级别和 sentence 级别两种准确率计算策略。函数处理了编码器 - 解码器结构、padding tokens 掩码等场景，用于评估生成式模型的预测准确率。

## 关键概念
- acc_strategy: 准确率计算策略，可选 token 级别或 sentence 级别
- 句子级别准确率: 句子中所有有效 token 都预测正确才算该句子正确
- Token 级别准确率: 分别计算每个有效 token 的预测准确率并求平均
- 掩码（mask）: 用于标识有效 token，-100 表示 padding 不参与计算

## 关联笔记
无
