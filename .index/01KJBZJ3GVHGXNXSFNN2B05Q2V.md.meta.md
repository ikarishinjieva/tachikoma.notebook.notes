---
note: 01KJBZJ3GVHGXNXSFNN2B05Q2V.md
title: 20241029 - 配置新的gpu服务器
indexed_at: 2026-02-27T11:20:14.241337+00:00
---

用户要求我分析一篇关于配置 GPU 服务器的笔记，并生成索引元数据。让我仔细分析笔记内容：

1. 笔记主题：配置新的 GPU 服务器
2. 主要内容：
   - 安装 Anaconda
   - 创建 conda 环境
   - SSH 代理配置
   - 配置 Llama-factory（LLM 训练框架）
   - 配置 clash 代理

3. 标签应该涵盖：GPU 服务器、环境配置、Llama-factory、代理配置、conda 等

4. 摘要：概括这是关于 GPU 服务器环境配置的笔记，包括 conda 环境、代理和 Llama-factory 的配置

5. 关键概念：conda 环境、SSH 反向代理、Llama-factory、系统代理、clash 等

6. 关联笔记：查看提供的笔记列表，主要是关于 MySQL、Clickhouse、数据库相关的笔记，与这篇 GPU 服务器配置笔记没有直接关联

让我按要求格式输出。
## 标签
GPU 服务器，环境配置，Llama-factory，代理配置，conda，深度学习

## 摘要
记录配置 GPU 服务器的完整流程，包括 Anaconda 安装、conda 环境创建、SSH 反向代理设置。重点配置 Llama-factory 训练框架和 clash 系统代理以便访问外部资源。

## 关键概念
- conda 环境：Python 包管理和隔离的虚拟环境工具
- SSH 反向代理：通过-R 参数将本地代理映射到远程服务器
- Llama-factory：大语言模型微调训练框架
- clash 代理：基于规则的代理工具，提供系统级网络代理
- 系统代理：通过环境变量 ALL_PROXY 配置全局代理

## 关联笔记
无
