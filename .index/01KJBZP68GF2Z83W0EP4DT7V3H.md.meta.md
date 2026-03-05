---
note: 01KJBZP68GF2Z83W0EP4DT7V3H.md
title: 20241224 - 阅读论文: CCOT: Compressed Chain of Thought: Efficient Reasoning through Dense Representations
indexed_at: 2026-03-05T11:07:25.949424+00:00
---

## 摘要
CCOT 提出将思维链推理逻辑压缩为向量表示，通过向量空间查询召回适用的推理逻辑并解码为文本，以加速 COT 推理过程。核心假设是相似推理过程对应向量空间中相近的点，该连续性通过训练隐式学习而非预先假设。

## 关键概念
- CCOT: 压缩思维链方法，将推理逻辑嵌入向量空间进行检索和生成
- 向量空间连续性: 相似推理过程在向量空间中对应相近位置的假设
- 隐藏状态近似: 从完整推理链中提取关键信息并压缩为向量表示
- 端到端优化: 通过反向传播学习将问题映射到合适向量表示

## 关联笔记
- 01KJBZSKZ0GFCFP50QGNNNFB3Y.md: PROMPTCOT 同样关注数学推理，但通过生成 rationale 合成训练数据而非压缩推理
- 01KJBZQZWKZXG432876BXTV83W.md: ZEUS 通过不确定性指导选择 few-shot 示例来增强零样本思维链推理
- 01KJBZQYYB3G1G5MW2TJ5MPTXW.md: CPO 使用 DPO 训练模型在思维链推理中选择更优的推理步骤
