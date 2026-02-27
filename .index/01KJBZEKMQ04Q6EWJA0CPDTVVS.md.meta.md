---
note: 01KJBZEKMQ04Q6EWJA0CPDTVVS.md
title: 20240823 - 解析 LaVague
indexed_at: 2026-02-27T11:13:19.851920+00:00
---

用户要求我分析一篇关于 LaVague 的笔记，并生成索引元数据。让我仔细阅读笔记内容：

1. 这是一篇关于 LaVague 框架的技术笔记
2. 主要内容包括：
   - WebAgent 的结构（WorldModel, ActionEngine）
   - NavigationEngine 的工作流程
   - Python Engine 的工作流程
   - NavigationControl
   - WORLD_MODEL_PROMPT_TEMPLATE

3. 核心主题是：LaVague 是一个基于 LLM 的 Web 自动化框架，使用多模态 LLM 分析网页截图，通过 NavigationEngine 和 PythonEngine 执行网页操作

4. 标签应该是：LaVague, Web 自动化，LLM, RAG, Selenium, 多模态

5. 关键概念：
   - WorldModel: 决策者，分析截图并思考下一步操作
   - NavigationEngine: 执行网页导航和操作
   - PythonEngine: 使用 LLM 分析 HTML 内容
   - ActionEngine: 管理其他 Engine，分配任务
   - Retriever: 检索与指令相关的 HTML 元素

6. 关联笔记：查看提供的笔记列表，都是关于 MySQL、Clickhouse、数据库、性能分析等主题，与 LaVague（Web 自动化/LLM 框架）没有明显关联，所以应该写「无」

让我按照要求的格式输出。## 标签
LaVague, Web 自动化，LLM, RAG, Selenium, 多模态

## 摘要
LaVague 是一个基于大语言模型的 Web 自动化框架，通过 WorldModel 进行高层决策，NavigationEngine 和 PythonEngine 执行具体网页操作。核心流程包括提取可操作 HTML 元素、RAG 检索相关上下文、LLM 生成决策并转换为 Selenium API 执行。

## 关键概念
- WorldModel: 决策者角色，分析浏览器截图并思考下一步操作的多模态 LLM 模块
- NavigationEngine: 执行网页导航操作，通过检索相关 HTML 元素并调用 LLM 生成动作
- PythonEngine: 使用 LLM 分析页面内容提取信息，支持可视区分析和全屏截图 fallback
- ActionEngine: 任务分配者，管理 NavigationEngine 和 PythonEngine 等子引擎
- Retriever: 从页面 HTML 中检索与指令相关的可操作元素，支持 XPath 和语义检索

## 关联笔记
无
