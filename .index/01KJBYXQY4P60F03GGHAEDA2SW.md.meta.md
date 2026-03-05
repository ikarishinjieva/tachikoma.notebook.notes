---
note: 01KJBYXQY4P60F03GGHAEDA2SW.md
title: 20230330 - llama-index 试用
indexed_at: 2026-03-05T08:29:39.162239+00:00
---

## 标签
llama-index, RAG, 环境配置, Jupyter, OpenAI, 向量检索

## 摘要
记录 llama-index 的 Docker 环境搭建、依赖安装及 Jupyter Notebook 配置流程。使用 MySQL manual 作为语料进行 RAG 检索测试，并给出输入输出示例。

## 关键概念
- llama-index: 用于构建 LLM 应用的 RAG 框架，支持多种索引结构和数据源
- RAG (Retrieval-Augmented Generation): 通过检索外部知识库增强 LLM 生成能力
- ALL_PROXY: 使用 socks5 代理访问 OpenAI API 的配置方式
- Azure OpenAI: llama-index 支持的 LLM 部署选项之一

## 关联笔记
- 01KJBZ044K0F6J2PM9E6FVJ07T.md: 分析 llama_index 为何不使用 refine 模式，与本篇同为 llama-index 技术探索
- 01KJBYYSWT0HJ1ZYSW82YFETXD.md: 介绍 LLM 知识获取架构，文中解决方案采用 llama-index 框架
- 01KJBZ52DM2FY8QMKH4WQ972HT.md: langchain 的 ConversationEntityMemory 分析，本篇提到 llama-index 可使用 langchain 定义的 LLM 模型
