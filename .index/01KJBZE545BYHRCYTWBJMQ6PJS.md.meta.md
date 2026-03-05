---
note: 01KJBZE545BYHRCYTWBJMQ6PJS.md
title: 20240812 - 试用AutoPrompt
indexed_at: 2026-03-05T10:31:36.277746+00:00
---

## 摘要
记录 AutoPrompt 项目的安装部署过程及遇到的各类兼容性问题。主要包括 Argilla API 升级导致的接口不一致、Azure OpenAI 模型 JSON 能力限制等问题，最终通过更换 GPT-4o 模型和修改源代码解决。

## 关键概念
- AutoPrompt: 基于 LLM 的自动化提示词优化框架，通过迭代生成测试样本和优化提示词
- Argilla: 开源的数据标注平台，用于 AutoPrompt 中的样本标注和评估
- Azure OpenAI: 微软 Azure 云服务提供的 OpenAI API，部分模型缺少 JSON 输出能力

## 关联笔记
- 01KJBZDYTJF2N82HT70Z06VNQN.md: AutoPrompt 架构分析笔记，记录了项目代码结构和运行流程
- 01KJBZEB61ZC33F8KAY5CR9GDJ.md: AutoPrompt 日志和流程分析，对运行日志进行了详细解析
- 01KJBZE0C1T7XST2M7PBZYEV9C.md: Intent-based Prompt Calibration 论文阅读，是 AutoPrompt 项目的理论基础
