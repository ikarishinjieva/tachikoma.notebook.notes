---
title: 20240723 - Dspy 学习
confluence_page_id: 3145823
created_at: 2024-07-23T05:28:42+00:00
updated_at: 2024-07-23T05:28:42+00:00
---

<https://mp.weixin.qq.com/s/yUL6eo_cio9mOkLhMW65wg>

其他提示词优化项目: 

  - gpt-prompt-engineer[6]：自动生成 prompt，LLM 运行之后生成结果。然后在 prompt 的生成 case 之间进行两两对战，裁判也是 LLM。最后通过胜负情况得出 ELO 分排名。总体逻辑比较简单，偏随机搜索。
  - AutoPrompt[7]：用户输入任务与初始 prompt。LLM 生成测试样例，由人工打标或者模型生成期望输出。接下来使用当前 prompt 进行预测，做结果分析，再进行 prompt 优化，不断循环提升。
  - SAMMO[8]：微软推出的 prompt 优化研究项目，粗看了下论文和代码，在 prompt 修改方面做的比较细致，包括重写，精简，删除，转换格式，从样例中生成指令等等。不过流行度和文档完善程度都不如 DSPy。
