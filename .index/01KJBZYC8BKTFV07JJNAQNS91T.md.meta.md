---
note: 01KJBZYC8BKTFV07JJNAQNS91T.md
title: 20250820 - 阅读论文*: Tracing the thoughts of a large language model (以生物学角度解释大模型是如何思考的)
indexed_at: 2026-03-05T11:54:43.019217+00:00
---

## 摘要
Anthropic 论文提出"AI 生物学"隐喻，将归因图作为"显微镜"系统解剖 LLM 内部工作机制。通过稀疏特征替代神经元、追踪因果联系、干预实验验证三步流程，揭示了模型内部的多步推理链、前瞻性规划、抽象泛化及安全拒绝等机制。

## 关键概念
- 归因图 (Attribution Graphs): 追踪特征间因果联系的可视化计算图谱，展示模型生成输出的中间步骤
- 特征 (Features): 稀疏激活且代表可解释概念的计算单元，用于替代原始模型的多义性神经元
- 超节点 (Supernodes): 将功能相似的特征组合成的简化节点，用于降低归因图复杂度
- 干预实验 (Intervention Experiments): 在原始模型中手动激活/抑制特征以验证因果关系的实验方法
- 跨层转码器 (CLT): 训练替代模型的工具，从原始模型神经元活动中学习可解释特征

## 关联笔记
- 01KJBZY0MAYHH592A9NG68XWV3.md: 同为 LLM 论文阅读笔记，关注小模型推理能力训练方法
- 01KJBZJG2DN96WB9H43JKKN8MY.md: 涉及使用 Claude 生成 CoT 思维链，与本笔记的推理追踪主题相关
