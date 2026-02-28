---
title: 20260228-材料研发AI
created_at: 2026-02-28T01:59:47.161843+00:00
updated_at: 2026-02-28T02:26:14.481412+00:00
---

# 背景

在春节期间, 做了一个材料研发的AI, 并在春节后与苏州国家实验室交流, 获得反馈

项目地址: <https://github.com/ikarishinjieva/mat_path.ai>

commit: c5ffb86a7f9d4df4f7bda89c08e7e24aa18d45a7

第一轮的输出结果: (待上传)

# 获得反馈

- 对于现在的目标指标( 击穿电压&gt; 65 kV, 闪点&gt; 110°C, 运动粘度 1.4 – 2.6 cSt, 溶胀 ≤ 纯正烷烃基准), 这些指标都是可以进行仿真计算的. 命名为 “筛选级关键指标预测器”
- 对于非常长的论文解读, 会有模型混淆知识的问题: <https://poe.com/s/xOgGWqsr4GCXgK4Rfvum>, 需要 对论文进行逻辑梳理, 以及引入事实性检查
- 非常好的一点, 是在需求分析环节, 识别出了 物理墙 (高闪点 和 低粘度 在分子原理上是冲突的)
- 但其选择初期筛选指标时, 是针对物理墙的冲突进行了选择, 但专家意见是: 需要以绝缘性作为初期筛选指标, 其在用户需求和物化本质两方面都属于 “可妥协空间很小”, 但闪点和粘度是在用户需求和物化本质两方面属于可调整/可妥协 空间比较大. 需要给模型这样的价值观, 来让模型通过一些材料研发论文, 来自己组织步骤

另外有几个参考工具, 可以作为资料来源: 

- scimaster (字节)

```
https://mail.126.com
邮箱：caizdchem@126.com
密码：CAIzhidao2025

https://scimaster.bohrium.com

先登录邮箱，然后进到scimaster的网站，选择邮箱登录，就可以在邮箱里收到验证码了
```

- google deepresearch
- openai research
- 书生的科学发现平台 MCP对接: <https://discovery.intern-ai.org.cn/home>
- <https://www.bohrium.com/>